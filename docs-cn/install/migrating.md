---
summary: "将 Moltbot 安装从一台机器迁移到另一台机器"
read_when:
  - 您正在将 Moltbot 移动到新笔记本电脑/服务器
  - 您想保留会话、身份验证和频道登录（WhatsApp 等）
---
# 将 Moltbot 迁移到新机器

本指南将 Moltbot 网关从一台机器迁移到另一台机器，**而无需重新进行入门**。

迁移在概念上很简单：

- 复制**状态目录**（`$CLAWDBOT_STATE_DIR`，默认：`~/.clawdbot/`）— 这包括配置、身份验证、会话和频道状态。
- 复制您的**工作区**（默认 `~/clawd/`）— 这包括您的代理文件（内存、提示等）。

但是在**配置文件**、**权限**和**部分复制**方面存在常见的陷阱。

## 开始之前（您正在迁移什么）

### 1) 识别您的状态目录

大多数安装使用默认值：

- **状态目录：** `~/.clawdbot/`

但如果您使用以下方式，可能会有所不同：

- `--profile <name>`（通常变为 `~/.clawdbot-<profile>/`）
- `CLAWDBOT_STATE_DIR=/some/path`

如果您不确定，请在**旧**机器上运行：

```bash
moltbot status
```

在输出中查找 `CLAWDBOT_STATE_DIR` / 配置文件的提及。如果您运行多个网关，请为每个配置文件重复此操作。

### 2) 识别您的工作区

常见默认值：

- `~/clawd/`（推荐工作区）
- 您创建的自定义文件夹

您的工作区是 `MEMORY.md`、`USER.md` 和 `memory/*.md` 等文件所在的位置。

### 3) 了解您将保留什么

如果您复制**状态目录和工作区两者**，您将保留：

- 网关配置（`moltbot.json`）
- 身份验证配置文件 / API 密钥 / OAuth 令牌
- 会话历史 + 代理状态
- 频道状态（例如 WhatsApp 登录/会话）
- 您的工作区文件（内存、技能笔记等）

如果您**仅**复制工作区（例如，通过 Git），您将**不会**保留：

- 会话
- 凭据
- 频道登录

这些位于 `$CLAWDBOT_STATE_DIR` 下。

## 迁移步骤（推荐）

### 步骤 0 — 进行备份（旧机器）

在**旧**机器上，首先停止网关，以免文件在复制过程中发生变化：

```bash
moltbot gateway stop
```

（可选但推荐）归档状态目录和工作区：

```bash
# 如果您使用配置文件或自定义位置，请调整路径
cd ~
tar -czf moltbot-state.tgz .clawdbot

tar -czf clawd-workspace.tgz clawd
```

如果您有多个配置文件/状态目录（例如 `~/.clawdbot-main`、`~/.clawdbot-work`），请分别归档每个目录。

### 步骤 1 — 在新机器上安装 Moltbot

在**新**机器上，安装 CLI（如果需要，还有 Node）：

- 参见：[安装](/install)

在此阶段，如果入门创建了新的 `~/.clawdbot/`，也没关系 — 您将在下一步覆盖它。

### 步骤 2 — 将状态目录 + 工作区复制到新机器

复制**两者**：

- `$CLAWDBOT_STATE_DIR`（默认 `~/.clawdbot/`）
- 您的工作区（默认 `~/clawd/`）

常见方法：

- 通过 `scp` 复制 tarball 并提取
- 通过 SSH 使用 `rsync -a`
- 外部驱动

复制后，确保：

- 包含了隐藏目录（例如 `.clawdbot/`）
- 文件所有权对于运行网关的用户是正确的

### 步骤 3 — 运行 Doctor（迁移 + 服务修复）

在**新**机器上：

```bash
moltbot doctor
```

Doctor 是"安全无聊"的命令。它会修复服务、应用配置迁移并警告不匹配。

然后：

```bash
moltbot gateway restart
moltbot status
```

## 常见陷阱（以及如何避免它们）

### 陷阱：配置文件 / 状态目录不匹配

如果您使用配置文件（或 `CLAWDBOT_STATE_DIR`）运行旧网关，而新网关使用不同的配置文件/状态目录，您将看到以下症状：

- 配置更改未生效
- 频道丢失 / 已登出
- 空的会话历史

修复：使用您迁移的**相同**配置文件/状态目录运行网关/服务，然后重新运行：

```bash
moltbot doctor
```

### 陷阱：仅复制 `moltbot.json`

`moltbot.json` 是不够的。许多提供商将状态存储在以下位置：

- `$CLAWDBOT_STATE_DIR/credentials/`
- `$CLAWDBOT_STATE_DIR/agents/<agentId>/...`

始终迁移整个 `$CLAWDBOT_STATE_DIR` 文件夹。

### 陷阱：权限 / 所有权

如果您以 root 身份复制或更改了用户，网关可能无法读取凭据/会话。

修复：确保状态目录 + 工作区由运行网关的用户拥有。

### 陷阱：在远程/本地模式之间迁移

- 如果您的 UI（WebUI/TUI）指向**远程**网关，远程主机拥有会话存储 + 工作区。
- 迁移笔记本电脑不会移动远程网关的状态。

如果您处于远程模式，请迁移**网关主机**。

### 陷阱：备份中的机密

`$CLAWDBOT_STATE_DIR` 包含机密（API 密钥、OAuth 令牌、WhatsApp 凭据）。像对待生产机密一样对待备份：

- 加密存储
- 避免通过不安全的渠道共享
- 如果怀疑泄露，请轮换密钥

## 验证清单

在新机器上，确认：

- `moltbot status` 显示网关正在运行
- 您的频道仍然已连接（例如，WhatsApp 不需要重新配对）
- 控制面板打开并显示现有会话
- 您的工作区文件（内存、配置）存在

## 相关文档

- [Doctor](/gateway/doctor)
- [网关故障排除](/gateway/troubleshooting)
- [Moltbot 在哪里存储其数据？](/help/faq#where-does-moltbot-store-its-data)
