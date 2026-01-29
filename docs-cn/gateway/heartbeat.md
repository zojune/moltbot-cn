---
summary: "心跳轮询消息和通知规则"
read_when:
  - 调整心跳节奏或消息传递
  - 在心跳和 cron 之间决定定期任务的使用
---
# 心跳(Gateway)

> **心跳还是 Cron?** 参见 [Cron vs Heartbeat](/automation/cron-vs-heartbeat) 了解何时使用每个的指导。

心跳在主会话中运行**定期代理轮次**,以便模型
可以显示需要注意的任何内容,而不会向您发送垃圾邮件。

## 快速入门(初学者)

1. 保持心跳启用(默认为 `30m`,对于 Anthropic OAuth/setup-token 为 `1h`)或设置您自己的节奏。
2. 在代理工作区中创建一个小的 `HEARTBEAT.md` 检查清单(可选但推荐)。
3. 决定心跳消息应该去哪里(`target: "last"` 是默认值)。
4. 可选:启用心跳推理传递以提高透明度。
5. 可选:将心跳限制在活动时间(本地时间)。

示例配置:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // 可选:也发送单独的 `Reasoning:` 消息
      }
    }
  }
}
```

## 默认值

- 间隔:`30m`(当 Anthropic OAuth/setup-token 是检测到的认证模式时为 `1h`)。设置 `agents.defaults.heartbeat.every` 或每个代理 `agents.list[].heartbeat.every`;使用 `0m` 禁用。
- 提示正文(可通过 `agents.defaults.heartbeat.prompt` 配置):
  `如果存在 HEARTBEAT.md(工作区上下文),请阅读它。严格遵循它。不要推断或重复先前聊天中的旧任务。如果不需要注意,请回复 HEARTBEAT_OK。`
- 心跳提示作为用户消息**逐字**发送。系统
  提示包括"Heartbeat"部分,并且运行在内部被标记。
- 活动时间(`heartbeat.activeHours`)在配置的时区中检查。
  在窗口之外,跳过心跳直到窗口内的下一个滴答。

## 心跳提示的用途

默认提示故意宽泛:
- **后台任务**:"考虑未完成的任务"促使代理审查
  后续工作(收件箱、日历、提醒、排队的工作)并显示任何紧急事项。
- **人工检查**:"有时在白天检查您的人类"促使偶尔的轻量级"您需要什么吗?"消息,但通过使用您配置的本地时区避免夜间垃圾邮件
  (参见 [/concepts/timezone](/concepts/timezone))。

如果您希望心跳执行非常具体的操作(例如"检查 Gmail PubSub
统计"或"验证 gateway 健康状况"),请将 `agents.defaults.heartbeat.prompt`(或
`agents.list[].heartbeat.prompt`)设置为自定义正文(逐字发送)。

## 响应约定

- 如果不需要注意,请回复 **`HEARTBEAT_OK`**。
- 在心跳运行期间,当 `HEARTBEAT_OK` 出现在回复的**开头或结尾**时,Moltbot 会将其视为确认。令牌将被剥离,如果剩余内容**≤ `ackMaxChars`**(默认:300),回复将被丢弃。
- 如果 `HEARTBEAT_OK` 出现在回复的**中间**,则不会对其进行特殊处理。
- 对于警报,**不要**包括 `HEARTBEAT_OK`;仅返回警报文本。

在心跳之外,消息开头/结尾的零散 `HEARTBEAT_OK` 被剥离
并记录;仅包含 `HEARTBEAT_OK` 的消息被丢弃。

## 配置

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",           // 默认:30m(0m 禁用)
        model: "anthropic/claude-opus-4-5",
        includeReasoning: false, // 默认:false(在可用时传递单独的 Reasoning: 消息)
        target: "last",         // last | none | <频道 id>(核心或插件,例如 "bluebubbles")
        to: "+15551234567",     // 可选的特定频道覆盖
        prompt: "如果存在 HEARTBEAT.md(工作区上下文),请阅读它。严格遵循它。不要推断或重复先前聊天中的旧任务。如果不需要注意,请回复 HEARTBEAT_OK。",
        ackMaxChars: 300         // HEARTBEAT_OK 之后允许的最大字符数
      }
    }
  }
}
```

### 作用域和优先级

- `agents.defaults.heartbeat` 设置全局心跳行为。
- `agents.list[].heartbeat` 在顶部合并;如果任何代理具有 `heartbeat` 块,**仅这些代理**运行心跳。
- `channels.defaults.heartbeat` 为所有频道设置可见性默认值。
- `channels.<channel>.heartbeat` 覆盖频道默认值。
- `channels.<channel>.accounts.<id>.heartbeat`(多帐户频道)覆盖每个频道设置。

### 每个代理的心跳

如果任何 `agents.list[]` 条目包含 `heartbeat` 块,**仅这些代理**
运行心跳。每个代理块在 `agents.defaults.heartbeat` 之上合并
(因此您可以设置一次共享默认值并按代理覆盖)。

示例:两个代理,仅第二个代理运行心跳。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last"
      }
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "如果存在 HEARTBEAT.md(工作区上下文),请阅读它。严格遵循它。不要推断或重复先前聊天中的旧任务。如果不需要注意,请回复 HEARTBEAT_OK。"
        }
      }
    ]
  }
}
```

### 字段说明

- `every`:心跳间隔(持续时间字符串;默认单位 = 分钟)。
- `model`:心跳运行的可选模型覆盖(`provider/model`)。
- `includeReasoning`:启用后,还会在可用时传递单独的 `Reasoning:` 消息(与 `/reasoning on` 形状相同)。
- `session`:心跳运行的可选会话密钥。
  - `main`(默认):代理主会话。
  - 显式会话密钥(从 `moltbot sessions --json` 或 [sessions CLI](/cli/sessions) 复制)。
  - 会话密钥格式:参见 [Sessions](/concepts/session) 和 [Groups](/concepts/groups)。
- `target`:
  - `last`(默认):传递到最后使用的外部频道。
  - 显式频道:`whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
  - `none`:运行心跳但**不传递**外部内容。
- `to`:可选的收件人覆盖(特定于频道的 id,例如 WhatsApp 的 E.164 或 Telegram 聊天 id)。
- `prompt`:覆盖默认提示正文(不合并)。
- `ackMaxChars`:在 `HEARTBEAT_OK` 之后传递之前允许的最大字符数。

## 传递行为

- 心跳默认在代理的主会话中运行(`agent:<id>:<mainKey>`),
  或在 `session.scope = "global"` 时运行 `global`。设置 `session` 覆盖到
  特定的频道会话(Discord/WhatsApp/等)。
- `session` 仅影响运行上下文;传递由 `target` 和 `to` 控制。
- 要传递到特定频道/收件人,请设置 `target` + `to`。使用
  `target: "last"`,传递使用该会话的最后一个外部频道。
- 如果主队列繁忙,心跳将被跳过并稍后重试。
- 如果 `target` 解析为没有外部目的地,运行仍会发生但不
  发送出站消息。
- 仅心跳的回复**不**保持会话活动;恢复最后的 `updatedAt`
  以使空闲过期正常工作。

## 可见性控制

默认情况下,`HEARTBEAT_OK` 确认被抑制,而警报内容被
传递。您可以按频道或按帐户调整此设置:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false      # 隐藏 HEARTBEAT_OK(默认)
      showAlerts: true   # 显示警报消息(默认)
      useIndicator: true # 发出指示器事件(默认)
  telegram:
    heartbeat:
      showOk: true       # 在 Telegram 上显示 OK 确认
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # 抑制此帐户的警报传递
```

优先级:每个帐户 → 每个频道 → 频道默认值 → 内置默认值。

### 每个标志的作用

- `showOk`:当模型返回仅 OK 的回复时发送 `HEARTBEAT_OK` 确认。
- `showAlerts`:当模型返回非 OK 的回复时发送警报内容。
- `useIndicator`:为 UI 状态表面发出指示器事件。

如果**所有三个**都为 false,Moltbot 将完全跳过心跳运行(无模型调用)。

### 每个频道与每个帐户的示例

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 所有 Slack 帐户
    accounts:
      ops:
        heartbeat:
          showAlerts: false # 仅抑制 ops 帐户的警报
  telegram:
    heartbeat:
      showOk: true
```

### 常见模式

| 目标 | 配置 |
| --- | --- |
| 默认行为(静默 OK,警报开启) | *(无需配置)* |
| 完全静默(无消息,无指示器) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| 仅指示器(无消息) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| 仅在一个频道中显示 OK | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md(可选)

如果工作区中存在 `HEARTBEAT.md` 文件,默认提示会告诉
代理阅读它。将其视为您的"心跳检查清单":小而稳定,并且
可以每 30 分钟安全地包含。

如果 `HEARTBEAT.md` 存在但实际上为空(只有空行和 markdown
标题,如 `# Heading`),Moltbot 会跳过心跳运行以节省 API 调用。
如果文件缺失,心跳仍会运行,模型决定要做什么。

保持其微小(短检查清单或提醒)以避免提示膨胀。

示例 `HEARTBEAT.md`:

```md
# 心跳检查清单

- 快速扫描:收件箱中有紧急内容吗?
- 如果是白天,如果没有其他待处理事项,请进行轻量级检查。
- 如果任务被阻止,请写下*缺少什么*并在下次询问 Peter。
```

### 代理可以更新 HEARTBEAT.md 吗?

可以 — 如果您要求它。

`HEARTBEAT.md` 只是代理工作区中的普通文件,因此您可以告诉
代理(在正常聊天中)类似以下内容:
- "更新 `HEARTBEAT.md` 以添加每日日历检查。"
- "重写 `HEARTBEAT.md` 使其更短并专注于收件箱后续工作。"

如果您希望主动执行此操作,您还可以在心跳提示中包含明确的行,
例如:"如果检查清单变得陈旧,请用更好的内容更新 HEARTBEAT_MD。"

安全说明:不要将机密(API 密钥、电话号码、私人令牌)放入
`HEARTBEAT.md` — 它将成为提示上下文的一部分。

## 手动唤醒(按需)

您可以使用以下命令将系统事件排队并触发立即心跳:

```bash
moltbot system event --text "检查紧急后续工作" --mode now
```

如果多个代理配置了 `heartbeat`,手动唤醒将立即运行每个代理的心跳。

使用 `--mode next-heartbeat` 等待下一个计划的滴答。

## 推理传递(可选)

默认情况下,心跳仅传递最终的"答案"负载。

如果您想要透明度,请启用:
- `agents.defaults.heartbeat.includeReasoning: true`

启用后,心跳还将传递单独的前缀为
`Reasoning:` 的消息(与 `/reasoning on` 形状相同)。当代理
管理多个会话/codex 并且您想查看它决定 ping 您的原因时,这很有用 — 但它也可能泄漏比您想要的更多内部细节。在群聊中最好保持关闭状态。

## 成本意识

心跳运行完整的代理轮次。较短的间隔会消耗更多令牌。保持
`HEARTBEAT.md` 较小,并考虑使用更便宜的 `model` 或 `target: "none"`,如果您
只想要内部状态更新。
