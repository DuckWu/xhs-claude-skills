---
name: xhs-batch
description: 批量提取小红书帖子，整理为本地 Markdown 知识库
user-invocable: true
argument-hint: <链接1> <链接2> ... 或粘贴多行链接
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, mcp__rednote__get_note_content, mcp__rednote__get_note_comments
---

用户希望批量提取多个小红书帖子。请按以下步骤处理：

## 常量定义
- Cookies 文件: `~/cookies.json`
- Obsidian 保存目录: `~/Documents/Obsidian Vault/xhs`
  - 若路径不存在，自动检测 Obsidian vault 位置
  - Windows 用户常见路径: `~/Documents/Obsidian/` 或自定义位置，请按实际情况修改
- Whisper 模型:
  - macOS: `mlx-community/whisper-large-v3-turbo`（使用 mlx-whisper）
  - Windows/Linux: `large-v3`（使用 openai-whisper）

## 输入
用户提供的链接列表: $ARGUMENTS

## 流程

### 步骤 1：解析链接
从输入中提取所有小红书链接（支持多行、空格分隔、逗号分隔）。
每个链接提取帖子 ID 和 xsec_token。

### 步骤 2：检查 Cookies
检查 `~/cookies.json` 是否存在。如不存在，按 `/xhs` 的步骤 0 引导用户导出。

### 步骤 3：逐个提取
对每个链接，执行 `/xhs` 的完整提取流程（步骤 2-4）：
- 请求页面 → 解析 __INITIAL_STATE__
- 视频帖子做语音转录
- 按 Peter Thiel 风格整理
- 保存为 `{YYYY-MM-DD} {短标题}.md`

每个帖子之间间隔 3 秒，避免触发反爬。

### 步骤 4：汇总报告
全部完成后，输出简短汇总：
- 成功/失败数量
- 每个帖子的文件名和一句话摘要
