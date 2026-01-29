---
summary: "子代理：生成隔离的代理运行，将结果公告回请求者聊天"
read_when:
  - 您希望通过代理进行后台/并行工作
  - 您正在更改 sessions_spawn 或子代理工具策略
---

# 子代理

子代理是从现有代理运行生成的后台代理运行。它们在自己的会话（`agent:<agentId>:subagent:<uuid>`）中运行，并在完成时将结果**公告**回请求者聊天频道。

## 斜杠命令

使用 `/subagents` 检查或控制**当前会话**的子代理运行：
- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` 显示运行元数据（状态、时间戳、会话 id、转录路径、清理）。

主要目标：
- 并行执行"研究 / 长任务 / 慢速工具"工作而不阻塞主运行。
- 默认保持子代理隔离（会话分离 + 可选沙箱）。
- 使工具界面难以误用：子代理**不会**默认获得会话工具。
- 避免嵌套扇出：子代理无法生成子代理。

成本说明：每个子代理都有其**自己的**上下文和 token 使用。对于繁重或重复的任务，为子代理设置更便宜的模型，并将主代理保持在更高质量的模型上。
您可以通过 `agents.defaults.subagents.model` 或每代理覆盖来配置此设置。

## 工具

使用 `sessions_spawn`：
- 启动子代理运行（`deliver: false`，全局通道：`subagent`）
- 然后运行公告步骤并将公告回复发布到请求者聊天频道
- 默认模型：继承调用者，除非您设置 `agents.defaults.subagents.model`（或每代理 `agents.list[].subagents.model`）；显式 `sessions_spawn.model` 仍然获胜。

工具参数：
- `task`（必需）
- `label?`（可选）
- `agentId?`（可选；如果允许，在另一个代理 id 下生成）
- `model?`（可选；覆盖子代理模型；无效值被跳过，子代理在默认模型上运行并在工具结果中发出警告）
- `thinking?`（可选；覆盖子代理运行的思考级别）
- `runTimeoutSeconds?`（默认 `0`；设置后，子代理运行在 N 秒后中止）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：
- `agents.list[].subagents.allowAgents`：可以通过 `agentId` 定向的代理 id 列表（`["*"]` 以允许任何）。默认：仅请求者代理。

发现：
- 使用 `agents_list` 查看当前允许用于 `sessions_spawn` 的代理 id。

自动归档：
- 子代理会话在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动归档。
- 归档使用 `sessions.delete` 并将转录重命名为 `*.deleted.<timestamp>`（同一文件夹）。
- `cleanup: "delete"` 在公告后立即归档（仍然通过重命名保留转录）。
- 自动归档是尽力而为；如果网关重启，挂起的计时器将丢失。
- `runTimeoutSeconds` **不会**自动归档；它仅停止运行。会话保持直到自动归档。

## 身份验证

子代理身份验证由**代理 id**解决，而不是会话类型：
- 子代理会话密钥是 `agent:<agentId>:subagent:<uuid>`。
- 身份验证存储从该代理的 `agentDir` 加载。
- 主代理的身份验证配置文件作为**回退**合并；代理配置文件在冲突时覆盖主配置文件。

注意：合并是累加的，因此主配置文件始终作为回退可用。尚未支持每个代理的完全隔离身份验证。

## 公告

子代理通过公告步骤报告：
- 公告步骤在子代理会话内运行（而不是请求者会话）。
- 如果子代理完全回复 `ANNOUNCE_SKIP`，则不发布任何内容。
- 否则，公告回复通过后续 `agent` 调用（`deliver=true`）发布到请求者聊天频道。
- 公告回复在可用时保留线程/主题路由（Slack 线程、Telegram 主题、Matrix 线程）。
- 公告消息被规范化为稳定的模板：
  - `Status:` 派生自运行结果（`success`、`error`、`timeout` 或 `unknown`）。
  - `Result:` 来自公告步骤的摘要内容（或如果缺失则为 `(not available)`）。
  - `Notes:` 错误详细信息和其他有用的上下文。
- `Status` 不是从模型输出推断的；它来自运行时结果信号。

公告载荷在末尾包括一个统计行（即使被包装）：
- 运行时（例如，`runtime 5m12s`）
- Token 使用（输入/输出/总计）
- 配置模型定价时的估计成本（`models.providers.*.models[].cost`）
- `sessionKey`、`sessionId` 和转录路径（以便主代理可以通过 `sessions_history` 获取历史记录或在磁盘上检查文件）

## 工具策略（子代理工具）

默认情况下，子代理获得**除会话工具外的所有工具**：
- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

通过配置覆盖：

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny 获胜
        deny: ["gateway", "cron"],
        // 如果设置了 allow，它将变为仅允许（deny 仍然获胜）
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```

## 并发

子代理使用专用的进程内队列通道：
- 通道名称：`subagent`
- 并发：`agents.defaults.subagents.maxConcurrent`（默认 `8`）

## 停止

- 在请求者聊天中发送 `/stop` 中止请求者会话并停止从中生成的任何活动子代理运行。

## 限制

- 子代理公告是**尽力而为**的。如果网关重启，挂起的"公告回"工作将丢失。
- 子代理仍然共享相同的网关进程资源；将 `maxConcurrent` 视为安全阀。
- `sessions_spawn` 始终是非阻塞的：它立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 子代理上下文仅注入 `AGENTS.md` + `TOOLS.md`（无 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
