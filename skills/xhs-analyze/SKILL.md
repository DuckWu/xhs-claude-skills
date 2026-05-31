---
name: xhs-analyze
description: 分析已提取的小红书收藏内容，做 AI 总结、对比、提炼
user-invocable: true
argument-hint: <关键词> 或留空分析全部
allowed-tools: Bash, Read, Glob, Grep
---

用户希望对已提取的小红书收藏做 AI 分析。

## 常量定义
- Obsidian 保存目录: `~/Documents/Obsidian Vault/xhs`
  - 若路径不存在，自动检测 Obsidian vault 位置
  - Windows 用户常见路径: `~/Documents/Obsidian/` 或自定义位置，请按实际情况修改

## 输入
用户查询: $ARGUMENTS

## 流程

### 步骤 1：加载数据
读取 `<Obsidian 保存目录>` 下所有 `.md` 文件（排除 img/video 子目录）。

如果用户指定了关键词，在文件名和内容中搜索匹配的帖子。
如果参数为空，读取全部帖子进行总览分析。

### 步骤 2：AI 分析
根据内容类型和用户意图，选择合适的分析方式：

**如果是教程/攻略类内容：**
- 提炼核心步骤为编号清单
- 如有多篇同类内容，做对比总结

**如果是知识/科普类内容：**
- 提炼关键知识点
- 整理为结构化笔记

**总览分析（无参数时）：**
- 收藏内容的主题分布
- 高频标签和关注领域
- 推荐可以深入整理的主题

### 步骤 3：输出格式
用清晰的中文 Markdown 格式输出分析结果。
