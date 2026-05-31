---
name: xhs
description: 提取小红书帖子内容（文字、图片、视频转录），整理为 Markdown 并保存
user-invocable: true
argument-hint: <小红书链接>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

用户希望提取小红书帖子内容。请按以下步骤处理：

## 常量定义
- Cookies 文件: `~/cookies.json`（从 Chrome 导出的小红书 cookies）
- Obsidian 保存目录: `~/Documents/Obsidian Vault/xhs`
  - 若路径不存在，自动检测 Obsidian vault 位置（搜索 `~/Documents/Obsidian`、`~/Documents/Obsidian Vault` 等常见路径）
  - Windows 用户常见路径: `~/Documents/Obsidian/` 或自定义位置，请按实际情况修改
- Whisper 模型:
  - macOS: `mlx-community/whisper-large-v3-turbo`（使用 mlx-whisper）
  - Windows/Linux: `large-v3`（使用 openai-whisper）

## 输入
用户提供的小红书链接: $ARGUMENTS

## 提取流程

### 步骤 0：检查 Cookies
1. 检查 `~/cookies.json` 是否存在
2. 如果不存在，告知用户需要从 Chrome 导出 cookies：
   - 在 Chrome 打开 xiaohongshu.com 并确认已登录
   - 打开 DevTools Console，运行以下代码将 cookies 复制到剪贴板：
   ```javascript
   copy(JSON.stringify(document.cookie.split('; ').map(c => {
     const [name, ...rest] = c.split('=');
     return { name, value: rest.join('='), domain: '.xiaohongshu.com', path: '/',
       expires: Date.now()/1000 + 86400*30, size: name.length + rest.join('=').length,
       httpOnly: false, secure: false, session: false, priority: 'Medium',
       sameParty: false, sourceScheme: 'Secure', sourcePort: 443 };
   })))
   ```
   - 将剪贴板内容保存到 `~/cookies.json`
   - 然后终止流程，等用户完成后重新运行

### 步骤 1：解析链接
从 URL 中提取帖子 ID（24 位十六进制字符串）和 xsec_token 参数。

### 步骤 2：获取帖子内容
使用 Python 脚本，通过 Cookies 请求帖子页面 HTML，从 `window.__INITIAL_STATE__` 解析全部帖子数据：

```python
import json, urllib.request, ssl, re

with open('<Cookies 文件>') as f:
    cookies = json.load(f)
cookie_str = '; '.join(f"{c['name']}={c['value']}" for c in cookies)

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

req = urllib.request.Request('<帖子URL>')
req.add_header('Cookie', cookie_str)
req.add_header('User-Agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36')

resp = urllib.request.urlopen(req, timeout=15, context=ctx)
html = resp.read().decode('utf-8', errors='ignore')

m = re.search(r'window\.__INITIAL_STATE__\s*=\s*(\{.+?\})\s*</script>', html, re.DOTALL)
raw = m.group(1).replace('undefined', 'null')
data = json.loads(raw)

# 帖子数据在: data['note']['noteDetailMap'][<key>]['note']
# 包含: title, desc, type, time, user, imageList, video, interactInfo, ipLocation
```

如果请求失败（被重定向到 404/错误页），说明 cookies 过期，提示用户按步骤 0 重新导出。

### 步骤 3：视频转录（仅视频帖子）
如果帖子 type 为 video，执行以下子步骤：

#### 3a. 提取视频 URL
从步骤 2 获取的数据中解析视频流：
```
note['video']['media']['stream'] -> 按 h264 > h265 > av1 优先级取第一个的 masterUrl
```

#### 3b. 下载视频并提取音频

先获取系统临时目录的真实路径（跨平台兼容）：

```python
import tempfile
print(tempfile.gettempdir())
```

将输出路径记为 `<TMP>`，然后：

```bash
# 下载视频
curl -L -o <TMP>/xhs_{post_id}.mp4 -H "Referer: https://www.xiaohongshu.com/" <视频URL>
# 提取音频
ffmpeg -y -i <TMP>/xhs_{post_id}.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 <TMP>/xhs_{post_id}.wav
```

> **Windows 注意**：如果 ffmpeg 不在当前会话的 PATH 中（winget 安装后需重启终端），先手动加入：
> ```bash
> # 用 where.exe ffmpeg 或 find 找到 ffmpeg.exe 的实际路径，然后 export
> ```

#### 3c. 语音转录

先检测平台，选择合适的 Whisper 实现。以下脚本自动处理跨平台路径、编码和 ffmpeg 查找：

```python
import platform, os, sys, tempfile

# 修复 Windows 中文输出乱码
if platform.system() == "Windows":
    sys.stdout.reconfigure(encoding='utf-8')

# 用真实系统临时目录（兼容 Git Bash /tmp/ 在子进程中无法解析的问题）
tmpdir = tempfile.gettempdir()
wav_path = os.path.join(tmpdir, "xhs_{post_id}.wav")

# Windows 下自动查找 ffmpeg 并加入 PATH（winget 安装后子进程可能找不到）
if platform.system() == "Windows":
    # 常见 ffmpeg 安装位置
    candidates = [
        os.path.join(os.environ.get("LOCALAPPDATA", ""),
            "Microsoft/WinGet/Packages/Gyan.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe"),
    ]
    for base in candidates:
        for root, dirs, files in os.walk(base):
            if "ffmpeg.exe" in files:
                os.environ["PATH"] = root + ";" + os.environ.get("PATH", "")
                break

if platform.system() == "Darwin":
    # macOS: 优先使用 mlx-whisper（Apple Silicon 原生优化，速度快）
    import mlx_whisper
    result = mlx_whisper.transcribe(wav_path,
        path_or_hf_repo="mlx-community/whisper-large-v3-turbo",
        language="zh", verbose=False)
    transcription = result["text"]
else:
    # Windows / Linux: 使用 openai-whisper（跨平台，需要 PyTorch）
    import whisper
    model = whisper.load_model("large-v3")
    result = model.transcribe(wav_path, language="zh")
    transcription = result["text"]
```

安装说明：
- **macOS**: `pip install mlx-whisper`
- **Windows / Linux**: `pip install openai-whisper`（首次运行会自动下载 large-v3 模型，约 3GB）
- **ffmpeg**（所有平台必需）：
  - macOS: `brew install ffmpeg`
  - Windows: `winget install ffmpeg` 或从 https://ffmpeg.org/download.html 下载
  - Linux: `sudo apt install ffmpeg`

#### 3d. 清理转录文本
- 去除尾部重复字符（背景音乐噪音）
- 按语义断句，添加标点和段落
- 如有步骤/要点结构，用 Markdown 格式化

#### 3e. 清理临时文件
```bash
rm -f <TMP>/xhs_{post_id}.mp4 <TMP>/xhs_{post_id}.wav
```

### 步骤 4：整理输出并保存
将内容整理为 Markdown 文件，保存到 `<Obsidian 保存目录>/{YYYY-MM-DD} {短标题}.md`。
- 文件名格式：`{发布日期} {短标题}.md`，短标题不超过15个字，是核心洞察的极简概括
- 日期前缀确保按时间排序
- 不创建子目录，所有帖子 md 直接放在 xhs 文件夹下
- 媒体文件统一放在 `<Obsidian 保存目录>/img/` 或 `<Obsidian 保存目录>/video/`

**写作风格：Peter Thiel 式——直接、反直觉、一句话给判断。笔记是决策工具，不是知识库。用户扫一眼就能决定：深挖还是跳过。**

文件结构（**无 YAML frontmatter**）：

```markdown
# 一句话核心洞察（反直觉的判断，不是描述性标题）

核心论点，2-3句话。直接给出"大多数人觉得X，但其实Y"的判断。
不废话，不铺垫，像 Thiel 在董事会上说话。

**与我的关联：** 一句话。读取用户的 memory（~/.claude/projects/*/memory/ 下的
user 和 project 类型记忆）了解用户背景、研究方向和当前工作，据此说清楚
这个内容跟用户有什么关系。如果 memory 不可用，从通用的个人发展/工具/方法论角度切入。

**值得深挖吗：** 是/否。一句话理由。

> [!tip]- 详情
> 帖子核心内容的结构化整理（折叠状态，点开才看到）：
> - 从 desc 和视频转录中提炼，清理 `#xxx[话题]#` 标记
> - 按逻辑结构分节，保留关键数据和结论
> - 图片用 `![图N](urlDefault)` 嵌入
> - 视频帖子在此处放整理后的转录内容

> [!info]- 笔记属性
> - **来源**: 小红书 · 作者名
> - **帖子ID**: xxx
> - **链接**: 原始链接
> - **日期**: YYYY-MM-DD
> - **类型**: image/video
> - **互动**: N赞 / N收藏 / N评论
> - **标签**: 标签1, 标签2, ...
```

关键约束：
- 折叠区域外的可见内容**不超过 6 行**
- 标题必须是洞察/判断，不是"XX帖子的总结"
- 图片使用 `urlDefault` 字段的 URL

### 步骤 5：更新索引笔记
在步骤 4 保存笔记后，更新 `<Obsidian 保存目录>/📋 索引.md`。这是一张扁平时间线——不做标签分组，标签作为搜索关键词内联在每条后面。

**索引文件结构：**

```markdown
# 📋 小红书收藏索引

> 按日期倒序。Cmd+F / Ctrl+F 搜标签即可定位。

- [[2026-05-14 短标题]] — 核心洞察 `#标签A #标签B`
- [[2026-05-10 另一个笔记]] — 核心洞察 `#标签C`
```

**更新逻辑：**

1. 用 Glob 列出 `xhs/` 下所有 `*.md` 文件（排除索引文件自身）
2. 对每个文件，用 Grep 提取行首的 `# ` 标题（核心洞察）和 `- **标签**:` 行里的标签列表
3. 按文件名中的日期前缀倒序排列（新在前）
4. 标签内联在行末，用反引号包裹，Obsidian 的全局搜索可命中
5. 用 Write 写入索引文件

**约束：**
- 不分组、不分级、不折叠——纯清单，长度不可控（笔记多了自然会很长）
- 不用 `## ` 二级标题做标签分组，标签只在行内出现
- 索引文件 `📋 索引.md` 本身不出现在列表中
