---
name: xhs-cover
description: 生成小红书封面图（HTML+CSS → Playwright 截图）
user-invocable: true
argument-hint: <标题文案> [--style 风格名]
allowed-tools: Bash, Read, Write, Edit
---

为用户生成一张小红书风格封面图。

## 输入
用户提供的内容: $ARGUMENTS

解析输入：
- 标题文案（必须）：从输入中提取核心文案，不超过 30 字
- 风格（可选）：`--style 风格名`，默认自动选择

## 可用风格

### 1. morandi（莫兰迪）
温柔高级，低饱和度，适合知识分享、方法论、生活方式
```
背景: linear-gradient(155deg, #f0e6d8, #e8dcd0, #d6cfc8)
主色: #c45c4a（暖红）
副色: #5b5080（灰紫）
辅助: #4a7a6a（灰绿）
文字: #3d3328（深棕）
标签背景: rgba(255,255,255,0.55)
```

### 2. academic（学术蓝）
冷静专业，适合论文解读、科研分享、技术分析
```
背景: linear-gradient(155deg, #e4eaf2, #d8e2ee, #cdd8e8)
主色: #2563eb（学术蓝）
副色: #7c3aed（紫）
辅助: #0891b2（青）
文字: #1e293b（深蓝灰）
标签背景: rgba(255,255,255,0.6)
```

### 3. dark（暗夜科技）
硬核科技感，适合 AI/编程/开源项目、技术深度内容
```
背景: linear-gradient(155deg, #1a1a2e, #16213e, #0f3460)
主色: #a78bfa（亮紫）
副色: #38bdf8（天蓝）
辅助: #34d399（绿）
文字: #f1f5f9（近白）
标签背景: rgba(255,255,255,0.08), border: rgba(255,255,255,0.2)
```

### 4. mint（薄荷清新）
清爽自然，适合工具推荐、效率方法、学习笔记
```
背景: linear-gradient(155deg, #ecfdf5, #e0f7ed, #d1f0e4)
主色: #059669（翠绿）
副色: #7c3aed（紫）
辅助: #d97706（琥珀）
文字: #1a3a2a（深绿灰）
标签背景: rgba(255,255,255,0.6)
```

### 5. sunset（日落暖橘）
温暖有活力，适合个人感悟、项目发布、里程碑分享
```
背景: linear-gradient(155deg, #fef3e2, #fde8d0, #fcdcc4)
主色: #ea580c（橘红）
副色: #7c3aed（紫）
辅助: #0891b2（青）
文字: #431407（深棕）
标签背景: rgba(255,255,255,0.55)
```

### 6. bw（极简黑白）
高级克制，适合观点输出、争议话题、深度思考
```
背景: linear-gradient(155deg, #f5f5f5, #ebebeb, #e0e0e0)
主色: #18181b（纯黑）
副色: #52525b（灰）
辅助: #a1a1aa（浅灰）
文字: #18181b（纯黑）
标签背景: rgba(255,255,255,0.7), border: #d4d4d8
```

## 生成流程

### 步骤 1：设计文案
从用户输入提取：
- **主标题**（最大字，1 行，核心概念/产品名/关键词）
- **副标题**（次大字，1-2 行，核心洞察或 tagline）
- **标签**（3 个功能/关键词标签，每个 2-4 字）
- **角标**（左上小字，身份标签如 OPEN SOURCE / 论文解读 / 工具推荐）
- **底部**（小字，3 个关键词用 · 分隔）

### 步骤 2：选择风格
- 如果用户指定了 `--style`，使用对应风格
- 否则根据内容自动选择：
  - 论文/科研 → academic
  - AI/编程/开源 → dark 或 morandi
  - 工具/效率 → mint
  - 个人感悟/发布 → sunset
  - 观点/思考 → bw

### 步骤 3：生成 HTML
写入 `/tmp/xhs-cover.html`，遵循以下模板结构：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { width: 1080px; height: 1440px; overflow: hidden;
         font-family: -apple-system, "PingFang SC", "Helvetica Neue", sans-serif; }
  .canvas {
    width: 1080px; height: 1440px;
    background: {风格背景};
    display: flex; flex-direction: column; justify-content: center;
    padding: 60px 70px; gap: 36px; position: relative;
  }
  /* 角标 */
  .badge { display: inline-flex; align-items: center; gap: 10px;
           background: {主色 alpha 0.12}; border: 1.5px solid {主色 alpha 0.2};
           border-radius: 10px; padding: 12px 24px; font-size: 28px;
           font-weight: 700; color: {主色}; letter-spacing: 3px; width: fit-content; }
  /* 主标题行 */
  .title-row { display: flex; align-items: baseline; gap: 20px; white-space: nowrap; }
  .title-main { font-size: 120-130px; font-weight: 900; color: {主色}; letter-spacing: -2px; }
  .title-secondary { font-size: 110-120px; font-weight: 900; color: {副色}; }
  /* 分隔线 */
  .divider { width: 80px; height: 5px;
             background: linear-gradient(90deg, {主色}, {副色});
             border-radius: 3px; margin: 10px 0; }
  /* 副标题 */
  .tagline { font-size: 76-82px; font-weight: 800; color: {文字色};
             letter-spacing: 8px; line-height: 1.4; }
  .tagline .accent { color: {主色 或副色 淡化}; }
  /* 标签 */
  .chip { background: {标签背景}; border-radius: 14px; padding: 18px 32px;
          font-size: 34px; font-weight: 700; letter-spacing: 2px; }
  /* 底部 */
  .footer { font-size: 30px; color: {文字色 淡化}; font-weight: 600; letter-spacing: 2px; }
</style>
</head>
<body>
  <!-- 按上面的结构填充内容 -->
</body>
</html>
```

关键设计约束：
- 3:4 比例（1080×1440px）
- 文字占画面 70% 以上
- 主标题尽量一行放下（必要时缩小字号或拆行）
- 2-3 级文字层级：主标题 > 副标题 > 标签/底部
- 所有中文用 font-weight: 700-900
- 颜色最多用 3 种（主+副+辅助），不杂乱

### 步骤 4：截图
用 Playwright 截图保存：

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(viewport={"width": 1080, "height": 1440})
    page.goto("file:///tmp/xhs-cover.html")
    page.wait_for_timeout(500)
    page.screenshot(path="<输出路径>", full_page=False)
    browser.close()
```

默认保存到 Obsidian vault 的 `xhs/img/cover-{简短标识}.png`（路径继承自 xhs skill 的常量定义，支持 macOS/Windows/Linux）。

### 步骤 5：展示
用 Read 工具展示生成的图片给用户，询问是否需要调整。
