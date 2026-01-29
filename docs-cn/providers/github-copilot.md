---
summary: "通过设备流程从 Moltbot 登录 GitHub Copilot"
read_when:
  - 你想使用 GitHub Copilot 作为模型提供程序
  - 你需要 `moltbot models auth login-github-copilot` 流程
---
# Github Copilot

## 什么是 GitHub Copilot？

GitHub Copilot 是 GitHub 的 AI 编码助手。它为你的 GitHub 账户和计划提供 Copilot 模型访问。Moltbot 可以通过两种不同方式将 Copilot 用作模型提供程序。

## 在 Moltbot 中使用 Copilot 的两种方式

### 1) 内置 GitHub Copilot 提供程序 (`github-copilot`)

使用本机设备登录流程获取 GitHub token，然后在 Moltbot 运行时将其交换为 Copilot API token。这是**默认**和最简单的方法，因为它不需要 VS Code。

### 2) Copilot Proxy 插件 (`copilot-proxy`)

使用 **Copilot Proxy** VS Code 扩展作为本地桥梁。Moltbot 与代理的 `/v1` 端点通信并使用你在那里配置的模型列表。当你已经在 VS Code 中运行 Copilot Proxy 或需要通过它路由时，选择此项。你必须启用插件并保持 VS Code 扩展运行。

使用 GitHub Copilot 作为模型提供程序 (`github-copilot`)。登录命令运行 GitHub 设备流程，保存身份验证配置文件，并更新你的配置以使用该配置文件。

## CLI 设置

```bash
moltbot models auth login-github-copilot
```

系统会提示你访问 URL 并输入一次性代码。保持终端打开直到完成。

### 可选标志

```bash
moltbot models auth login-github-copilot --profile-id github-copilot:work
moltbot models auth login-github-copilot --yes
```

## 设置默认模型

```bash
moltbot models set github-copilot/gpt-4o
```

### 配置片段

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```

## 注意事项

- 需要交互式 TTY；直接在终端中运行它。
- Copilot 模型可用性取决于你的计划；如果模型被拒绝，请尝试另一个 ID（例如 `github-copilot/gpt-4.1`）。
- 登录在身份验证配置文件存储中存储 GitHub token，并在 Moltbot 运行时将其交换为 Copilot API token。
