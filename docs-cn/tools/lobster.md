---
title: Lobster
summary: "Moltbot 的类型化工作流运行时，具有可恢复的批准网关。"
description: Moltbot 的类型化工作流运行时 — 具有批准网关的可组合管道。
read_when:
  - 您想要具有明确批准的确定性多步骤工作流
  - 您需要恢复工作流而不重新运行早期步骤
---

# Lobster

Lobster 是一个工作流 shell，让 Moltbot 将多步骤工具序列作为单一的确定性操作运行，并具有明确的批准检查点。

## 钩子

您的助手可以构建管理自己的工具。要求一个工作流，30 分钟后您就拥有了一个 CLI 加上作为一个调用运行的管道。Lobster 是缺失的部分：确定性管道、明确的批准和可恢复状态。

## 为什么

今天，复杂的工作流需要许多来回的工具调用。每次调用都会消耗 token，LLM 必须编排每一步。Lobster 将编排移到了类型化运行时中：

- **一次调用而不是多次**：Moltbot 运行一次 Lobster 工具调用并获得结构化结果。
- **内置批准**：副作用（发送电子邮件、发布评论）暂停工作流，直到明确批准。
- **可恢复**：暂停的工作流返回一个令牌；批准并恢复而无需重新运行所有内容。

## 为什么是 DSL 而不是普通程序？

Lobster 故意很小。目标不是"一种新语言"，而是一个可预测的、AI 友好的管道规范，具有一流的批准和恢复令牌。

- **内置批准/恢复**：普通程序可以提示人类，但它不能*暂停和恢复*而持久的令牌，除非您自己发明该运行时。
- **确定性 + 可审计性**：管道是数据，因此易于记录、差异、重放和审查。
- **AI 的约束表面**：微小的语法 + JSON 管道减少了"创造性"代码路径，并使验证现实。
- ** baked-in 安全策略**：超时、输出上限、沙箱检查和允许列表由运行时强制执行，而不是每个脚本。
- **仍然可编程**：每一步都可以调用任何 CLI 或脚本。如果您想要 JS/TS，请从代码生成 `.lobster` 文件。

## 工作原理

Moltbot 在**工具模式**下启动本地 `lobster` CLI 并从 stdout 解析 JSON 信封。如果管道暂停以获得批准，工具返回 `resumeToken`，以便您稍后继续。

## 模式：小型 CLI + JSON 管道 + 批准

构建说话 JSON 的小命令，然后将它们链接到单个 Lobster 调用中。（下面的示例命令名称 — 交换您自己的）。

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

如果管道请求批准，请使用令牌恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI 触发工作流；Lobster 执行步骤。批准网关使副作用明确且可审计。

示例：将输入项映射到工具调用：

```bash
gog.gmail.search --query 'newer_than:1d' \
  | clawd.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## 仅限 JSON 的 LLM 步骤（llm-task）

对于需要**结构化 LLM 步骤**的工作流，启用可选的 `llm-task` 插件工具并从 Lobster 调用它。这使工作流保持确定性，同时仍然允许您使用模型进行分类/总结/起草。

启用工具：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

在管道中使用它：

```lobster
clawd.invoke --tool llm-task --action json --args-json '{
  "prompt": "给定输入电子邮件，返回意图和草稿。",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

请参阅 [LLM 任务](/tools/llm-task)了解详细信息和配置选项。

## 工作流文件（.lobster）

Lobster 可以运行带有 `name`、`args`、`steps`、`env`、`condition` 和 `approval` 字段的 YAML/JSON 工作流文件。在 Moltbot 工具调用中，将 `pipeline` 设置为文件路径。

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

说明：

- `stdin: $step.stdout` 和 `stdin: $step.json` 传递先前步骤的输出。
- `condition`（或 `when`）可以基于 `$step.approved` 限制步骤。

## 安装 Lobster

在运行 Moltbot 网关的**同一主机**上安装 Lobster CLI（请参阅 [Lobster 仓库](https://github.com/moltbot/lobster)），并确保 `lobster` 在 `PATH` 上。如果您想使用自定义二进制位置，请在工具调用中传递**绝对** `lobsterPath`。

## 启用工具

Lobster 是一个**可选**插件工具（默认未启用）。

推荐（累加、安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

或每代理：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

除非您打算以限制性允许列表模式运行，否则避免使用 `tools.allow: ["lobster"]`。

注意：允许列表是可选插件的加入。如果您的允许列表仅命名插件工具（如 `lobster`），Moltbot 保持核心工具启用。要限制核心工具，请将您想要的核心工具或组也包含在允许列表中。

## 示例：电子邮件分类

没有 Lobster：
```
用户："检查我的电子邮件并起草回复"
→ clawd 调用 gmail.list
→ LLM 总结
  用户："为 #2 和 #5 起草回复"
→ LLM 起草
  用户："发送 #2"
→ clawd 调用 gmail.send
（每天重复，没有已分类内容的记忆）
```

使用 Lobster：
```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

返回 JSON 信封（截断）：
```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 需要回复，2 需要操作" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "发送 2 个草稿回复？",
    "items": [],
    "resumeToken": "..."
  }
}
```

用户批准 → 恢复：
```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

一个工作流。确定性。安全。

## 工具参数

### `run`

在工具模式下运行管道。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

使用参数运行工作流文件：

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### `resume`

在批准后继续暂停的工作流。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### 可选输入

- `lobsterPath`：Lobster 二进制文件的绝对路径（省略以使用 `PATH`）。
- `cwd`：管道的工作目录（默认为当前进程工作目录）。
- `timeoutMs`：如果超过此持续时间则终止子进程（默认：20000）。
- `maxStdoutBytes`：如果 stdout 超过此大小则终止子进程（默认：512000）。
- `argsJson`：传递给 `lobster run --args-json` 的 JSON 字符串（仅工作流文件）。

## 输出信封

Lobster 返回具有以下三种状态之一的 JSON 信封：

- `ok` → 成功完成
- `needs_approval` → 暂停；需要 `requiresApproval.resumeToken` 来恢复
- `cancelled` → 明确拒绝或取消

工具在 `content`（漂亮 JSON）和 `details`（原始对象）中都公开信封。

## 批准

如果存在 `requiresApproval`，请检查提示并决定：

- `approve: true` → 恢复并继续副作用
- `approve: false` → 取消并完成工作流

使用 `approve --preview-from-stdin --limit N` 将 JSON 预览附加到批准请求，而无需自定义 jq/heredoc 粘合。恢复令牌现在很紧凑：Lobster 在其状态目录下存储工作流恢复状态并返回一个小令牌密钥。

## OpenProse

OpenProse 与 Lobster 配对良好：使用 `/prose` 编排多代理准备，然后运行 Lobster 管道以进行确定性批准。如果 Prose 程序需要 Lobster，请通过 `tools.subagents.tools` 为子代理允许 `lobster` 工具。请参阅 [OpenProse](/prose)。

## 安全性

- **仅本地子进程** — 插件本身没有网络调用。
- **无密钥** — Lobster 不管理 OAuth；它调用管理 OAuth 的 clawd 工具。
- **感知沙箱** — 当工具上下文处于沙箱状态时禁用。
- **加固** — 如果指定，`lobsterPath` 必须是绝对的；强制执行超时和输出上限。

## 故障排除

- **`lobster subprocess timed out`** → 增加 `timeoutMs`，或拆分长管道。
- **`lobster output exceeded maxStdoutBytes`** → 提高 `maxStdoutBytes` 或减少输出大小。
- **`lobster returned invalid JSON`** → 确保管道在工具模式下运行并仅打印 JSON。
- **`lobster failed (code …)`** → 在终端中运行相同的管道以检查 stderr。

## 了解更多

- [插件](/plugin)
- [插件工具创作](/plugins/agent-tools)

## 案例研究：社区工作流

一个公开示例："第二大脑" CLI + 管理三个 Markdown 保管库（个人、合作伙伴、共享）的 Lobster 管道。CLI 发出 JSON 用于统计、收件箱列表和陈旧扫描；Lobster 将这些命令链接到工作流中，如 `weekly-review`、`inbox-triage`、`memory-consolidation` 和 `shared-task-sync`，每个工作流都有批准网关。AI 在可用时处理判断（分类），否则回退到确定性规则。

- 线程：https://x.com/plattenschieber/status/2014508656335770033
- 仓库：https://github.com/bloomedai/brain-cli
