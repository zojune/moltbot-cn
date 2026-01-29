---
summary: "网关调度器的 Cron 任务和唤醒功能"
read_when:
  - 调度后台任务或唤醒
  - 连接应与心跳一起或 alongside 心跳运行的自动化
  - 在心跳和 cron 之间为调度任务做决定
---
# Cron 任务（网关调度器）

> **Cron 与 Heartbeat 的区别？** 请参阅 [Cron 与 Heartbeat](/automation/cron-vs-heartbeat) 了解何时使用哪个的指导。

Cron 是网关的内置调度器。它持久化任务，在正确的时间唤醒代理，并可以选择将输出传回聊天。

如果您想要*"每天早上运行这个"*或*"20 分钟后提醒代理"*，
cron 就是您需要的机制。

## 简而言之
- Cron 在**网关内部**运行（不在模型内部）。
- 任务持久化在 `~/.clawdbot/cron/` 下，因此重启不会丢失调度。
- 两种执行方式：
  - **主会话**：将系统事件加入队列，然后在下一次心跳时运行。
  - **隔离**：在 `cron:<jobId>` 中运行专用的代理轮次，可选择传递输出。
- 唤醒是一等公民：任务可以请求"立即唤醒"与"下一次心跳"。

## 新手友好的概述
将 cron 任务视为：**何时**运行 + **做什么**。

1) **选择调度方式**
   - 一次性提醒 → `schedule.kind = "at"` (CLI: `--at`)
   - 重复任务 → `schedule.kind = "every"` 或 `schedule.kind = "cron"`
   - 如果您的 ISO 时间戳省略时区，将被视为 **UTC**。

2) **选择运行位置**
   - `sessionTarget: "main"` → 在下一次心跳期间使用主上下文运行。
   - `sessionTarget: "isolated"` → 在 `cron:<jobId>` 中运行专用的代理轮次。

3) **选择有效负载**
   - 主会话 → `payload.kind = "systemEvent"`
   - 隔离会话 → `payload.kind = "agentTurn"`

可选：`deleteAfterRun: true` 在成功运行一次性任务后从存储中删除该任务。

## 概念

### 任务
一个 cron 任务是一个存储的记录，包含：
- 一个**调度**（何时运行），
- 一个**有效负载**（做什么），
- 可选的**传递**（输出应发送到哪里）。
- 可选的**代理绑定**（`agentId`）：在特定代理下运行任务；如果
  缺失或未知，网关将回退到默认代理。

任务由稳定的 `jobId` 标识（由 CLI/网关 API 使用）。
在代理工具调用中，`jobId` 是规范的；为了兼容性接受旧版 `id`。
任务可以通过 `deleteAfterRun: true` 在成功运行一次性任务后自动删除。

### 调度
Cron 支持三种调度类型：
- `at`：一次性时间戳（自纪元以来的毫秒数）。网关接受 ISO 8601 并强制转换为 UTC。
- `every`：固定间隔（毫秒）。
- `cron`：5 字段 cron 表达式，可选 IANA 时区。

Cron 表达式使用 `croner`。如果省略时区，则使用网关主机的本地时区。

### 主会话与隔离执行

#### 主会话任务（系统事件）
主任务将系统事件加入队列，并可选择唤醒心跳运行器。
它们必须使用 `payload.kind = "systemEvent"`。

- `wakeMode: "next-heartbeat"`（默认）：事件等待下一次调度的心跳。
- `wakeMode: "now"`：事件立即触发心跳运行。

当您想要正常的心跳提示 + 主会话上下文时，这是最佳选择。
请参阅 [Heartbeat](/gateway/heartbeat)。

#### 隔离任务（专用 cron 会话）
隔离任务在会话 `cron:<jobId>` 中运行专用的代理轮次。

关键行为：
- 提示以 `[cron:<jobId> <job name>]` 为前缀，以便于追踪。
- 每次运行开始一个新的**会话 id**（没有先前的对话延续）。
- 摘要被发布到主会话（前缀 `Cron`，可配置）。
- `wakeMode: "now"` 在发布摘要后立即触发心跳。
- 如果 `payload.deliver: true`，输出被传递到频道；否则它保持内部。

将隔离任务用于不应污染主聊天历史记录的嘈杂、频繁或"后台杂务"。

### 有效负载形状（运行内容）
支持两种有效负载类型：
- `systemEvent`：仅主会话，通过心跳提示路由。
- `agentTurn`：仅隔离会话，运行专用的代理轮次。

常见的 `agentTurn` 字段：
- `message`：必需的文本提示。
- `model` / `thinking`：可选覆盖（见下文）。
- `timeoutSeconds`：可选超时覆盖。
- `deliver`：`true` 将输出发送到频道目标。
- `channel`：`last` 或特定频道。
- `to`：频道特定的目标（电话/聊天/频道 id）。
- `bestEffortDeliver`：如果传递失败，避免使任务失败。

隔离选项（仅适用于 `session=isolated`）：
- `postToMainPrefix` (CLI: `--post-prefix`)：主会话中系统事件的前缀。
- `postToMainMode`：`summary`（默认）或 `full`。
- `postToMainMaxChars`：当 `postToMainMode=full` 时的最大字符数（默认 8000）。

### 模型和思考覆盖
隔离任务（`agentTurn`）可以覆盖模型和思考级别：
- `model`：提供商/模型字符串（例如 `anthropic/claude-sonnet-4-20250514`）或别名（例如 `opus`）
- `thinking`：思考级别（`off`、`minimal`、`low`、`medium`、`high`、`xhigh`；仅限 GPT-5.2 + Codex 模型）

注意：您也可以在主会话任务上设置 `model`，但这会改变共享的主会话模型。
我们建议仅对隔离任务使用模型覆盖，以避免意外的上下文转换。

解析优先级：
1. 任务有效负载覆盖（最高）
2. 钩子特定的默认值（例如 `hooks.gmail.model`）
3. 代理配置默认值

### 传递（频道 + 目标）
隔离任务可以将输出发送到频道。任务有效负载可以指定：
- `channel`：`whatsapp` / `telegram` / `discord` / `slack` / `mattermost`（插件）/ `signal` / `imessage` / `last`
- `to`：频道特定的接收者目标

如果省略 `channel` 或 `to`，cron 可以回退到主会话的"最后路由"（代理上次回复的地方）。

传递说明：
- 如果设置了 `to`，即使省略了 `deliver`，cron 也会自动传递代理的最终输出。
- 当您想要没有显式 `to` 的最后路由传递时，使用 `deliver: true`。
- 使用 `deliver: false` 使输出保持内部，即使存在 `to`。

目标格式提醒：
- Slack/Discord/Mattermost（插件）目标应使用显式前缀（例如 `channel:<id>`、`user:<id>`）以避免歧义。
- Telegram 主题应使用 `:topic:` 格式（见下文）。

#### Telegram 传递目标（主题 / 论坛主题）
Telegram 通过 `message_thread_id` 支持论坛主题。对于 cron 传递，您可以将主题/主题编码到 `to` 字段中：

- `-1001234567890`（仅聊天 id）
- `-1001234567890:topic:123`（推荐：显式主题标记）
- `-1001234567890:123`（简写：数字后缀）

也接受像 `telegram:...` / `telegram:group:...` 这样的前缀目标：
- `telegram:group:-1001234567890:topic:123`

## 存储和历史
- 任务存储：`~/.clawdbot/cron/jobs.json`（网关管理的 JSON）。
- 运行历史：`~/.clawdbot/cron/runs/<jobId>.jsonl`（JSONL，自动修剪）。
- 覆盖存储路径：配置中的 `cron.store`。

## 配置

```json5
{
  cron: {
    enabled: true, // 默认 true
    store: "~/.clawdbot/cron/jobs.json",
    maxConcurrentRuns: 1 // 默认 1
  }
}
```

完全禁用 cron：
- `cron.enabled: false`（配置）
- `CLAWDBOT_SKIP_CRON=1`（环境变量）

## CLI 快速入门

一次性提醒（UTC ISO，成功后自动删除）：
```bash
moltbot cron add \
  --name "发送提醒" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "提醒：提交费用报告。" \
  --wake now \
  --delete-after-run
```

一次性提醒（主会话，立即唤醒）：
```bash
moltbot cron add \
  --name "日历检查" \
  --at "20m" \
  --session main \
  --system-event "下一次心跳：检查日历。" \
  --wake now
```

重复隔离任务（传递到 WhatsApp）：
```bash
moltbot cron add \
  --name "早晨状态" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "总结今天的收件箱 + 日历。" \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

重复隔离任务（传递到 Telegram 主题）：
```bash
moltbot cron add \
  --name "夜间摘要（主题）" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "总结今天；发送到夜间主题。" \
  --deliver \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

具有模型和思考覆盖的隔离任务：
```bash
moltbot cron add \
  --name "深度分析" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "每周深度分析项目进度。" \
  --model "opus" \
  --thinking high \
  --deliver \
  --channel whatsapp \
  --to "+15551234567"
```

代理选择（多代理设置）：
```bash
# 将任务固定到代理 "ops"（如果该代理缺失，则回退到默认）
moltbot cron add --name "运维检查" --cron "0 6 * * *" --session isolated --message "检查运维队列" --agent ops

# 切换或清除现有任务上的代理
moltbot cron edit <jobId> --agent ops
moltbot cron edit <jobId> --clear-agent
```
```

手动运行（调试）：
```bash
moltbot cron run <jobId> --force
```

编辑现有任务（补丁字段）：
```bash
moltbot cron edit <jobId> \
  --message "更新的提示" \
  --model "opus" \
  --thinking low
```

运行历史：
```bash
moltbot cron runs --id <jobId> --limit 50
```

不创建任务的即时系统事件：
```bash
moltbot system event --mode now --text "下一次心跳：检查电池。"
```

## 网关 API 表面
- `cron.list`、`cron.status`、`cron.add`、`cron.update`、`cron.remove`
- `cron.run`（强制或到期）、`cron.runs`
对于没有任务的即时系统事件，请使用 [`moltbot system event`](/cli/system)。

## 故障排除

### "没有运行任何东西"
- 检查 cron 是否已启用：`cron.enabled` 和 `CLAWDBOT_SKIP_CRON`。
- 检查网关是否持续运行（cron 在网关进程内运行）。
- 对于 `cron` 调度：确认时区（`--tz`）与主机时区。

### Telegram 传递到错误的位置
- 对于论坛主题，使用 `-100…:topic:<id>`，使其明确且无歧义。
- 如果您在日志或存储的"最后路由"目标中看到 `telegram:...` 前缀，那是正常的；
  cron 传递接受它们并仍然正确解析主题 ID。
