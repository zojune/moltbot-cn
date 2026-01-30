# Moltbot ACP 网桥

本文档描述了 Moltbot ACP（Agent 客户端协议）网桥的工作原理，
它如何将 ACP 会话映射到 Gateway 会话，以及 IDE 应该如何调用它。

## 概述

`moltbot acp` 通过 stdio 公开 ACP agent，并将提示转发到运行的
Moltbot Gateway 通过 WebSocket。它保持 ACP 会话 id 映射到 Gateway
会话密钥，以便 IDE 可以重新连接到相同的 agent 脚本或根据请求重置它。

主要目标：

- 最小的 ACP 表面积（stdio、NDJSON）。
- 跨重新连接的稳定会话映射。
- 使用现有的 Gateway 会话存储（list/resolve/reset）。
- 安全默认值（默认情况下隔离的 ACP 会话密钥）。

## 如何使用这个

当 IDE 或工具使用 Agent 客户端协议并且你希望它
驱动 Moltbot Gateway 会话时，请使用 ACP。

快速步骤：

1. 运行 Gateway（本地或远程）。
2. 配置 Gateway 目标（`gateway.remote.url` + auth）或传递标志。
3. 将 IDE 指向通过 stdio 运行 `moltbot acp`。

示例配置：

```bash
moltbot config set gateway.remote.url wss://gateway-host:18789
moltbot config set gateway.remote.token <token>
```

示例运行：

```bash
moltbot acp --url wss://gateway-host:18789 --token <token>
```

## 选择 agents

ACP 不直接选择 agents。它通过 Gateway 会话密钥路由。

使用 agent 作用域的会话密钥来定位特定的 agent：

```bash
moltbot acp --session agent:main:main
moltbot acp --session agent:design:main
moltbot acp --session agent:qa:bug-123
```

每个 ACP 会话映射到单个 Gateway 会话密钥。一个 agent 可以拥有许多
会话；除非你覆盖密钥或标签，否则 ACP 默认为隔离的 `acp:<uuid>` 会话。

## Zed 编辑器设置

在 `~/.config/zed/settings.json` 中添加自定义 ACP agent：

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

要定位特定的 Gateway 或 agent：

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

在 Zed 中，打开 Agent 面板并选择"Moltbot ACP"以启动线程。

## 执行模型

- ACP 客户端生成 `moltbot acp` 并通过 stdio 传递 ACP 消息。
- 网桥使用现有的 auth 配置（或 CLI 标志）连接到 Gateway。
- ACP `prompt` 转换为 Gateway `chat.send`。
- Gateway 流式事件被转换回 ACP 流式事件。
- ACP `cancel` 映射到活动运行的 Gateway `chat.abort`。

## 会话映射

默认情况下，每个 ACP 会话映射到专用的 Gateway 会话密钥：

- `acp:<uuid>`，除非被覆盖。

你可以通过两种方式覆盖或重用会话：

1) CLI 默认值

```bash
moltbot acp --session agent:main:main
moltbot acp --session-label "support inbox"
moltbot acp --reset-session
```

2) 每个会话的 ACP 元数据

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true,
    "requireExisting": false
  }
}
```

规则：

- `sessionKey`：直接 Gateway 会话密钥。
- `sessionLabel`：通过标签解析现有会话。
- `resetSession`：在首次使用之前为密钥铸造新的脚本。
- `requireExisting`：如果密钥/标签不存在则失败。

### 会话列表

ACP `listSessions` 映射到 Gateway `sessions.list` 并返回过滤的
摘要，适用于 IDE 会话选择器。`_meta.limit` 可以限制返回的
会话数量。

## 提示转换

ACP 提示输入被转换为 Gateway `chat.send`：

- `text` 和 `resource` 块成为提示文本。
- 带有图像 mime 类型的 `resource_link` 成为附件。
- 工作目录可以前缀到提示中（默认开启，可以通过 `--no-prefix-cwd` 禁用）。

Gateway 流式事件被转换为 ACP `message` 和 `tool_call`
更新。终端 Gateway 状态映射到带有停止原因的 ACP `done`：

- `complete` -> `stop`
- `aborted` -> `cancel`
- `error` -> `error`

## Auth + Gateway 发现

`moltbot acp` 从 CLI 标志或配置解析 Gateway URL 和 auth：

- `--url` / `--token` / `--password` 优先。
- 否则使用配置的 `gateway.remote.*` 设置。

## 操作说明

- ACP 会话在网桥进程生命周期内存储在内存中。
- Gateway 会话状态由 Gateway 本身持久化。
- `--verbose` 将 ACP/Gateway 网桥事件记录到 stderr（绝不是 stdout）。
- ACP 运行可以被取消，并且每个会话跟踪活动运行 id。

## 兼容性

- ACP 网桥使用 `@agentclientprotocol/sdk`（目前 0.13.x）。
- 适用于实现 `initialize`、`newSession`、
  `loadSession`、`prompt`、`cancel` 和 `listSessions` 的 ACP 客户端。

## 测试

- 单元：`src/acp/session.test.ts` 覆盖运行 id 生命周期。
- 完整门控：`pnpm lint && pnpm build && pnpm test && pnpm docs:build`。

## 相关文档

- CLI 用法：`docs/cli/acp.md`
- 会话模型：`docs/concepts/session.md`
- 会话管理内部：`docs/reference/session-management-compaction.md`
