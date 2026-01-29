# 创建自定义技能 🛠

Moltbot 旨在易于扩展。"技能"是为助手添加新功能的主要方式。

## 什么是技能？

技能是一个包含 `SKILL.md` 文件（为 LLM 提供说明和工具定义）的目录，以及可选的一些脚本或资源。

## 逐步进行：您的第一个技能

### 1. 创建目录

技能位于您的工作区中，通常位于 `~/clawd/skills/`。为您的技能创建一个新文件夹：
```bash
mkdir -p ~/clawd/skills/hello-world
```

### 2. 定义 `SKILL.md`

在该目录中创建 `SKILL.md` 文件。此文件使用 YAML frontmatter 作为元数据，使用 Markdown 作为说明。

```markdown
---
name: hello_world
description: 一个简单的打招呼技能。
---

# Hello World 技能
当用户问好时，使用 `echo` 工具说"Hello from your custom skill!"。
```

### 3. 添加工具（可选）

您可以在 frontmatter 中定义自定义工具，或指示代理使用现有的系统工具（如 `bash` 或 `browser`）。

### 4. 刷新 Moltbot

让您的代理"刷新技能"或重启网关。Moltbot 将发现新目录并索引 `SKILL.md`。

## 最佳实践
- **简洁**：指示模型做*什么*，而不是如何成为 AI。
- **安全第一**：如果您的技能使用 `bash`，请确保提示不允许来自不受信任的用户输入的任意命令注入。
- **本地测试**：使用 `moltbot agent --message "use my new skill"` 进行测试。

## 共享技能

您还可以浏览和贡献技能到 [ClawdHub](https://clawdhub.com)。
