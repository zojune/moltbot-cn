---
summary: "运行 ACP 网桥以支持 IDE 集成"
read_when:
  - 设置基于 ACP 的 IDE 集成
  - 调试 ACP 会话到 Gateway 的路由
---

# acp

运行 ACP (Agent Client Protocol) 网桥,该网桥与 Moltbot Gateway 通信。

此命令通过 stdio 使用 ACP 与 IDE 通信,并通过 WebSocket 将提示转发给 Gateway。它保持 ACP 会话映射到 Gateway 会话密钥。

## 用法

```bash
moltbot acp

# 远程 Gateway
moltbot acp --url wss://gateway-host:18789 --token <token>

# 附加到现有会话密钥
moltbot acp --session agent:main:main

# 通过标签附加(必须已存在)
moltbot acp --session-label "support inbox"

# 在第一个提示之前重置会话密钥
moltbot acp --session agent:main:main --reset-session
```

## ACP 客户端(调试)

使用内置的 ACP 客户端来检查网桥的正常性,而无需 IDE。它会生成 ACP 网桥并允许您交互式地输入提示。

```bash
moltbot acp client

# 将生成的网桥指向远程 Gateway
moltbot acp client --server-args --url wss://gateway-host:18789 --token <token>

# 覆盖服务器命令(默认:moltbot)
moltbot acp client --server "node" --server-args moltbot.mjs acp --url ws://127.0.0.1:19001
```

## 如何使用

当 IDE(或其他客户端)使用 Agent Client Protocol,并且您希望它驱动 Moltbot Gateway 会话时,请使用 ACP。

1. 确保 Gateway 正在运行(本地或远程)。
2. 配置 Gateway 目标(配置或标志)。
3. 将您的 IDE 指向通过 stdio 运行 `moltbot acp`。

示例配置(持久化):

```bash
moltbot config set gateway.remote.url wss://gateway-host:18789
moltbot config set gateway.remote.token <token>
```

示例直接运行(无配置写入):

```bash
moltbot acp --url wss://gateway-host:18789 --token <token>
```

## 选择 agent

ACP 不直接选择 agent。它通过 Gateway 会话密钥进行路由。

使用 agent 作用域的会话密钥来定位特定的 agent:

```bash
moltbot acp --session agent:main:main
moltbot acp --session agent:design:main
moltbot acp --session agent:qa:bug-123
```

每个 ACP 会话映射到单个 Gateway 会话密钥。一个 agent 可以拥有多个会话;除非您覆盖密钥或标签,否则 ACP 默认为隔离的 `acp:<uuid>` 会话。

## Zed 编辑器设置

在 `~/.config/zed/settings.json` 中添加自定义 ACP agent(或使用 Zed 的设置 UI):

```json
{
  "agent_servers": {
    "Moltbot ACP": {
      "type": "custom",
      "command": "moltbot",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

要定位特定的 Gateway 或 agent:

```json
{
  "agent_servers": {
    "Moltbot ACP": {
      "type": "custom",
      "command": "moltbot",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

在 Zed 中,打开 Agent 面板并选择 "Moltbot ACP" 以启动对话。

## 会话映射

默认情况下,ACP 会话获得一个带有 `acp:` 前缀的隔离 Gateway 会话密钥。要重用已知会话,请传递会话密钥或标签:

- `--session <key>`: 使用特定的 Gateway 会话密钥。
- `--session-label <label>`: 通过标签解析现有会话。
- `--reset-session`: 为该密钥生成新的会话 id(相同的密钥,新的对话)。

如果您的 ACP 客户端支持元数据,您可以按会话覆盖:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

在 [/concepts/session](/concepts/session) 了解更多关于会话密钥的信息。

## 选项

- `--url <url>`: Gateway WebSocket URL(默认为配置中的 gateway.remote.url)。
- `--token <token>`: Gateway 认证令牌。
- `--password <password>`: Gateway 认证密码。
- `--session <key>`: 默认会话密钥。
- `--session-label <label>`: 要解析的默认会话标签。
- `--require-existing`: 如果会话密钥/标签不存在则失败。
- `--reset-session`: 在首次使用之前重置会话密钥。
- `--no-prefix-cwd`: 不在提示前添加工作目录前缀。
- `--verbose, -v`: 详细日志输出到 stderr。

### `acp client` 选项

- `--cwd <dir>`: ACP 会话的工作目录。
- `--server <command>`: ACP 服务器命令(默认: `moltbot`)。
- `--server-args <args...>`: 传递给 ACP 服务器的额外参数。
- `--server-verbose`: 在 ACP 服务器上启用详细日志记录。
- `--verbose, -v`: 详细客户端日志。
