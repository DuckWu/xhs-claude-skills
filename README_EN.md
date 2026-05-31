<div align="center">

# 📕 RedNote to Obsidian

[![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-7C3AED)](https://docs.anthropic.com/en/docs/claude-code)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**[中文](README.md)** &nbsp;|&nbsp; English

</div>

---

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that extracts [RedNote (小红书)](https://www.xiaohongshu.com) posts into concise [Obsidian](https://obsidian.md) notes. Supports text, images, and video — video posts are automatically downloaded and transcribed locally with whisper.

Core mode uses only cookies + HTTP + local models. Optionally integrate [rednote MCP](https://github.com/DuckWu/rednote-mcp) to fetch comments and auto-select the Top 10 most-liked for your notes.

---

## 🚀 Install

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (runtime environment)
- [Obsidian](https://obsidian.md) (just needs the vault folder — no CLI required)
- Video transcription (optional):
  - **macOS**: `brew install ffmpeg` + `pip install mlx-whisper`
  - **Windows**: `winget install ffmpeg` (**restart terminal** after install) + `pip install openai-whisper` (first run downloads ~3GB large-v3 model)
  - **Linux**: `sudo apt install ffmpeg` + `pip install openai-whisper`

### Install the plugin

```bash
/plugin marketplace add DuckWu/xhs-claude-skills
/plugin install rednote-to-obsidian@DuckWu-xhs-claude-skills
```

### First use

```
/xhs https://www.xiaohongshu.com/explore/...
```

On first run, the skill auto-guides you through a **30-second cookie setup**:

1. 🌐 Open Chrome → xiaohongshu.com (make sure you're logged in)
2. 🔧 Open DevTools Console (F12)
3. 📋 Paste the snippet the skill gives you → cookies auto-copied to clipboard
4. 💾 Save to `~/cookies.json`
5. ✅ Done — all future runs use this automatically

> 🔄 When cookies expire, the skill detects it and re-prompts. No manual checking needed.

---

## ✨ Features

| Command | Description |
|:--------|:------------|
| `/xhs <url>` | 📄 Extract a single post — text, images, video transcription |
| `/xhs-mcp <url>` | 💬 Extract post + MCP top comments (requires rednote MCP) |
| `/xhs-batch <urls>` | 📦 Batch extract multiple posts |
| `/xhs-analyze [keyword]` | 🔍 Analyze saved posts — summarize, compare, find patterns |

> 💡 The default `/xhs` does not fetch comments. To enable comments, install [rednote MCP](https://github.com/DuckWu/rednote-mcp), log in, then use `/xhs-mcp`. All other features work without MCP.
>
> ⚠️ **Known issue**: Some `rednote.com` links may return 404 in MCP. Workaround: manually replace `rednote.com` with `xiaohongshu.com` in the link (the post ID stays the same).

### 📂 Output

Notes are date-sorted and saved directly in your Obsidian vault under `xhs/`:

```
xhs/
├── 2026-03-22 YY-methodology.md
├── 2026-03-29 ZZ-breakthrough.md
├── img/
└── video/
```

Each note is a **decision tool** — scan in 5 seconds, decide to dig deeper or skip:

```markdown
# One-line insight                     ← judgment, not description

Core argument, 2-3 sentences.

**Relevance:** Why this matters to you.
**Worth digging?** Yes/No + reason.

> [!tip]- Details                       ← collapsed by default
> Structured content...

> [!info]- Metadata                     ← collapsed by default
> Source · date · stats · tags
```

The "Relevance" line reads from Claude Code's [memory system](https://docs.anthropic.com/en/docs/claude-code) to auto-adapt to your background. No manual config needed.

---

## 🏗 How it works

```
 RedNote URL
     │
     ▼
 ┌─────────────────────────┐
 │  Cookie auth              │  ← Reuses Chrome login session
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  Parse __INITIAL_STATE__ │  ← One HTTP request, all data
 └────┬──────┬──────┬──────┘
      ▼      ▼      ▼
    Text   Images  Video
     │              │
     │         curl → ffmpeg → whisper transcribe
     │              │
     ▼              ▼
 ┌─────────────────────────┐
 │  ✨ Peter Thiel format    │  ← Insight-driven, collapsible
 └────────────┬────────────┘
              ▼
        Obsidian note
```

> 💡 Use `/xhs-mcp <url>` to add MCP comment fetching to the above flow. Comments appear as Top 10 most-liked section in the note.

---

## ⚙️ Configuration

| Setting | Default | Description |
|:--------|:--------|:------------|
| Cookies | `~/cookies.json` | RedNote auth |
| Output dir | `~/Documents/Obsidian Vault/xhs` | Obsidian vault path. Windows users often use `~/Documents/Obsidian/`. Edit constants in `skills/xhs/SKILL.md` if your path differs. |

Platform-specific defaults:
- **macOS**: `~/Documents/Obsidian Vault/xhs`
- **Windows**: `~/Documents/Obsidian/xhs` (adjust if different)
- **Linux**: `~/Documents/Obsidian Vault/xhs`

## 📁 Plugin structure

```
rednote-to-obsidian/
├── .claude-plugin/plugin.json
└── skills/
    ├── xhs/SKILL.md
    ├── xhs-mcp/SKILL.md
    ├── xhs-batch/SKILL.md
    └── xhs-analyze/SKILL.md
```

<div align="center">

---

Forked from [chenxiachan/xhs-claude-skills](https://github.com/chenxiachan/xhs-claude-skills). MIT License

</div>
