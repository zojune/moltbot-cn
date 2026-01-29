---
summary: "代理会话工具用于列出会话、获取历史记录和发送跨会话消息"
read_when:
  - 添加或修改会话工具
---

# 会话工具

目标：小型、难以误用的工具集，以便代理可以列出会话、获取历史记录和发送到另一个会话。

## 工具名称
- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

## 键模型
- 主直接聊天桶始终是字面键 `"main"`（解析为当前代理的主键）。
- 群组聊天使用 `agent:<agentId>:<channel>:group:<id>` 或 `agent:<agentId>:<channel>:channel:<id>`（传递完整键）。
- Cron 作业使用 `cron:<job.id>`。
- Hooks 使用 `hook:<uuid>`，除非明确设置。
- 节点会话使用 `node-<nodeId>`，除非明确设置。

`global` 和 `unknown` 是保留值，永远不会列出。如果 `session.scope = "global"`，我们会为所有工具将其别名化为 `main`，以便调用者永远看不到 `global`。

## sessions_list
将会话列为行数组。

参数：
- `kinds?: string[]` 过滤器：`"main" | "group" | "cron" | "hook" | "node" | "other"` 中的任何一个
- `limit?: number` 最大行数（默认：服务器默认值，例如限制为 200）
- `activeMinutes?: number` 仅在 N 分钟内更新的会话
- `messageLimit?: number` 0 = 无消息（默认 0）；>0 = 包含最后 N 条消息

行为：
- `messageLimit > 0` 为每个会话获取 `chat.history` 并包含最后 N 条消息。
- 工具结果在列表输出中被过滤；使用 `sessions_history` 获取工具消息。
- 在**沙箱化**代理会话中运行时，会话工具默认为**仅生成可见性**（见下文）。

行形状（JSON）：
- `key`：会话键（字符串）
- `kind`：`main | group | cron | hook | node | other`
- `channel`：`whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName`（群组显示标签，如果可用）
- `updatedAt`（ms）
- `sessionId`
- `model`、`contextTokens`、`totalTokens`
- `thinkingLevel`、`verboseLevel`、`systemSent`、`abortedLastRun`
- `sendPolicy`（会话覆盖，如果设置）
- `lastChannel`、`lastTo`
- `deliveryContext`（可用时的规范化 `{ channel, to, accountId }`）
- `transcriptPath`（从存储目录 + sessionId 派生的尽力而为路径）
- `messages?`（仅当 `messageLimit > 0` 时）

## sessions_history
获取一个会话的记录。

参数：
- `sessionKey`（必需；接受来自 `sessions_list` 的会话键或 `sessionId`）
- `limit?: number` 最大消息数（服务器限制）
- `includeTools?: boolean`（默认 false）

行为：
- `includeTools=false` 过滤 `role: "toolResult"` 消息。
- 以原始记录格式返回消息数组。
- 当给定 `sessionId` 时，Moltbot 将其解析为相应的会话键（缺失的 ID 错误）。

## sessions_send
向另一个会话发送消息。

参数：
- `sessionKey`（必需；接受来自 `sessions_list` 的会话键或 `sessionId`）
- `message`（必需）
- `timeoutSeconds?: number`（默认 >0；0 = 即发即弃）

行为：
- `timeoutSeconds = 0`：排队并返回 `{ runId, status: "accepted" }`。
- `timeoutSeconds > 0`：等待最多 N 秒完成，然后返回 `{ runId, status: "ok", reply }`。
- 如果等待超时：`{ runId, status: "timeout", error }`。运行继续；稍后调用 `sessions_history`。
- 如果运行失败：`{ runId, status: "error", error }`。
- 公告交付运行在主运行完成后运行并且是尽力而为的；`status: "ok"` 不保证公告已交付。
- 通过网关 `agent.wait` 等待（服务器端），因此重新连接不会中断等待。
- 为主运行注入代理到代理消息上下文。
- 主运行完成后，Moltbot 运行**回复回循环**：
  - 第 2+ 轮在请求者和目标代理之间交替。
  - 准确回复 `REPLY_SKIP` 以停止乒乓。
  - 最大轮次为 `session.agentToAgent.maxPingPongTurns`（0–5，默认 5）。
- 循环结束后，Moltbot 运行**代理到代理公告步骤**（仅目标代理）：
  - 准确回复 `ANNOUNCE_SKIP` 以保持沉默。
  - 任何其他回复发送到目标通道。
  - 公告步骤包括原始请求 + 第 1 轮回复 + 最新乒乓回复。

## 通道字段
- 对于群组，`channel` 是会话条目上记录的通道。
- 对于直接聊天，`channel` 从 `lastChannel` 映射。
- 对于 cron/hook/node，`channel` 是 `internal`。
- 如果缺失，`channel` 是 `unknown`。

## 安全 / 发送策略
基于通道/聊天类型的基于策略的阻止（不是每个会话 ID）。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

运行时覆盖（每个会话条目）：
- `sendPolicy: "allow" | "deny"`（未设置 = 继承配置）
- 可通过 `sessions.patch` 或仅所有者的 `/send on|off|inherit`（独立消息）设置。

执行点：
- `chat.send` / `agent`（网关）
- 自动回复交付逻辑

## sessions_spawn
在隔离会话中生成子代理运行并将结果公告回请求者聊天通道。

参数：
- `task`（必需）
- `label?`（可选；用于日志/UI）
- `agentId?`（可选；如果允许，在另一个代理 ID 下生成）
- `model?`（可选；覆盖子代理模型；无效值错误）
- `runTimeoutSeconds?`（默认 0；设置时，在 N 秒后中止子代理运行）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：
- `agents.list[].subagents.allowAgents`：允许通过 `agentId` 的代理 ID 列表（`["*"]` 以允许任何）。默认：仅请求者代理。

发现：
- 使用 `agents_list` 发现哪些代理 ID 允许用于 `sessions_spawn`。

行为：
- 启动一个新的 `agent:<agentId>:subagent:<uuid>` 会话，并设置 `deliver: false`。
- 子代理默认为完整工具集**减去会话工具**（可通过 `tools.subagents.tools` 配置）。
- 子代理不允许调用 `sessions_spawn`（无子代理 → 子代理生成）。
- 始终非阻塞：立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 完成后，Moltbot 运行子代理**公告步骤**并将结果发布到请求者聊天通道。
- 在公告步骤期间准确回复 `ANNOUNCE_SKIP` 以保持沉默。
- 公告回复规范化为 `Status`/`Result`/`Notes`；`Status` 来自运行时结果（而不是模型文本）。
- 子代理会话在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动存档。
- 公告回复包括统计行（运行时、令牌、sessionKey/sessionId、记录路径和可选成本）。

## 沙箱会话可见性

沙箱化会话可以使用会话工具，但默认情况下它们只能看到通过 `sessions_spawn` 生成的会话。

配置：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // 默认："spawned"
        sessionToolsVisibility: "spawned" // 或 "all"
      }
    }
  }
}
```
