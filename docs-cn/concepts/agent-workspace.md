---
summary: "Agent 工作区：位置、布局和备份策略"
read_when:
  - 您需要解释 agent 工作区或其文件布局
  - 您想备份或迁移 agent 工作区
---
# Agent 工作区

工作区是 agent 的家。它是用于文件工具和工作区上下文的唯一工作目录。保持其私密并将其视为记忆。

这与 `~/.clawdbot/` 分开，后者存储配置、凭据和会话。

**重要提示：**工作区是**默认 cwd**，而不是硬沙盒。工具根据工作区解析相对路径，但绝对路径仍可到达主机上的其他位置，除非启用沙盒。如果您需要隔离，请使用 [`agents.defaults.sandbox`](/gateway/sandboxing)（和/或每个 agent 的沙盒配置）。
当启用沙盒且 `workspaceAccess` 不是 `"rw"` 时，工具在 `~/.clawdbot/sandboxes` 下的沙盒工作区内操作，而不是您的主机工作区。

## 默认位置

- 默认值：`~/clawd`
- 如果设置了 `CLAWDBOT_PROFILE` 且不是 `"default"`，默认值变为
  `~/clawd-<profile>`。
- 在 `~/.clawdbot/moltbot.json` 中覆盖：

```json5
{
  agent: {
    workspace: "~/clawd"
  }
}
```

`moltbot onboard`、`moltbot configure` 或 `moltbot setup` 将创建工作区并在缺失时播种引导文件。

如果您已经自己管理工作区文件，可以禁用引导文件创建：

```json5
{ agent: { skipBootstrap: true } }
```

## 额外的工作区文件夹

较旧的安装可能创建了 `~/moltbot`。保留多个工作区目录可能会导致身份验证或状态漂移的混淆，因为一次只有一个工作区处于活动状态。

**推荐：**保持单个活动工作区。如果您不再使用额外的文件夹，请将它们存档或移动到废纸篓（例如 `trash ~/moltbot`）。
如果您有意保留多个工作区，请确保 `agents.defaults.workspace` 指向活动的工作区。

`moltbot doctor` 在检测到额外的工作区目录时会发出警告。

## 工作区文件映射（每个文件的含义）

这些是 Moltbot 在工作区内期望的标准文件：

- `AGENTS.md`
  - Agent 的操作说明以及它应该如何使用记忆。
  - 在每次会话开始时加载。
  - 规则、优先级和"如何行为"细节的好地方。

- `SOUL.md`
  - 人格、语气和边界。
  - 每次会话加载。

- `USER.md`
  - 用户是谁以及如何称呼他们。
  - 每次会话加载。

- `IDENTITY.md`
  - Agent 的名称、氛围和表情符号。
  - 在引导仪式期间创建/更新。

- `TOOLS.md`
  - 关于本地工具和约定的注释。
  - 不控制工具可用性；它只是指导。

- `HEARTBEAT.md`
  - 用于心跳运行的可选小清单。
  - 保持简短以避免令牌消耗。

- `BOOT.md`
  - 可选启动清单，在启用内部挂钩时在 gateway 重启时执行。
  - 保持简短；使用消息工具进行出站发送。

- `BOOTSTRAP.md`
  - 一次性首次运行仪式。
  - 仅为全新的工作区创建。
  - 在仪式完成后删除。

- `memory/YYYY-MM-DD.md`
  - 每日记忆日志（每天一个文件）。
  - 建议在会话开始时读取今天 + 昨天。

- `MEMORY.md`（可选）
  - 策划的长期记忆。
  - 仅在主、私密会话中加载（不在共享/组上下文中）。

请参阅 [Memory](/concepts/memory) 了解工作流和自动记忆刷新。

- `skills/`（可选）
  - 工作区特定的 skills。
  - 当名称冲突时覆盖托管/捆绑的 skills。

- `canvas/`（可选）
  - 用于节点显示的 Canvas UI 文件（例如 `canvas/index.html`）。

如果任何引导文件缺失，Moltbot 会在会话中注入"缺失文件"标记并继续。大型引导文件在注入时被截断；通过 `agents.defaults.bootstrapMaxChars` 调整限制（默认：20000）。
`moltbot setup` 可以重新创建缺失的默认值而不覆盖现有文件。

## 工作区中没有的内容

这些位于 `~/.clawdbot/` 下，不应提交到工作区仓库：

- `~/.clawdbot/moltbot.json`（配置）
- `~/.clawdbot/credentials/`（OAuth 令牌、API 密钥）
- `~/.clawdbot/agents/<agentId>/sessions/`（会话记录 + 元数据）
- `~/.clawdbot/skills/`（托管的 skills）

如果您需要迁移会话或配置，请单独复制它们并将它们排除在版本控制之外。

## Git 备份（推荐，私密）

将工作区视为私密记忆。将其放在**私有** git 仓库中，以便备份和恢复。

在运行 Gateway 的机器上运行这些步骤（即工作区所在的位置）。

### 1) 初始化仓库

如果安装了 git，全新的工作区会自动初始化。如果此工作区还不是仓库，请运行：

```bash
cd ~/clawd
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) 添加私有远程存储（对初学者友好的选项）

选项 A：GitHub Web UI

1. 在 GitHub 上创建一个新的**私有**仓库。
2. 不要使用 README 初始化（避免合并冲突）。
3. 复制 HTTPS 远程 URL。
4. 添加远程并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

选项 B：GitHub CLI (`gh`)

```bash
gh auth login
gh repo create clawd-workspace --private --source . --remote origin --push
```

选项 C：GitLab Web UI

1. 在 GitLab 上创建一个新的**私有**仓库。
2. 不要使用 README 初始化（避免合并冲突）。
3. 复制 HTTPS 远程 URL。
4. 添加远程并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) 持续更新

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## 不要提交机密

即使在私有仓库中，也要避免在工作区中存储机密：

- API 密钥、OAuth 令牌、密码或私密凭据。
- `~/.clawdbot/` 下的任何内容。
- 聊天的原始转储或敏感附件。

如果您必须存储敏感引用，请使用占位符并将真正的机密保留在其他地方（密码管理器、环境变量或 `~/.clawdbot/`）。

建议的 `.gitignore` 入门：

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## 将工作区移动到新机器

1. 将仓库克隆到所需路径（默认 `~/clawd`）。
2. 在 `~/.clawdbot/moltbot.json` 中将 `agents.defaults.workspace` 设置为该路径。
3. 运行 `moltbot setup --workspace <path>` 以播种任何缺失的文件。
4. 如果需要会话，请从旧机器单独复制 `~/.clawdbot/agents/<agentId>/sessions/`。

## 高级说明

- 多 agent 路由可以为每个 agent 使用不同的工作区。请参阅
  [Channel routing](/concepts/channel-routing) 了解路由配置。
- 如果启用了 `agents.defaults.sandbox`，非 main 会话可以使用 `agents.defaults.sandbox.workspaceRoot` 下的每个会话沙盒工作区。
