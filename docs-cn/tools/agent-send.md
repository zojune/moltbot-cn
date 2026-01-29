---
summary: "直接 `moltbot agent` CLI 运行（可选投递）"
read_when:
  - 添加或修改代理 CLI 入口点
---
# `moltbot agent`（直接代理运行）

`moltbot agent` 运行单个代理轮次，无需接收入站聊天消息。
默认情况下它**通过网关**运行；添加 `--local` 可强制在当前机器上使用嵌入式运行时。

## 行为

- 必需：`--message <text>`
- 会话选择：
  - `--to <dest>` 派生会话密钥（群组/频道目标保持隔离；直接聊天折叠为 `main`），**或**
  - `--session-id <id>` 通过 id 重用现有会话，**或**
  - `--agent <id>` 直接定向到已配置的代理（使用该代理的 `main` 会话密钥）
- 运行与正常入站回复相同的嵌入式代理运行时。
- 思考/详细标志会持久化到会话存储中。
- 输出：
  - 默认：打印回复文本（加上 `MEDIA:<url>` 行）
  - `--json`：打印结构化载荷 + 元数据
- 可选投递到频道，使用 `--deliver` + `--channel`（目标格式匹配 `moltbot message --target`）。
- 使用 `--reply-channel`/`--reply-to`/`--reply-account` 覆盖投递而不更改会话。

如果网关不可达，CLI **回退**到嵌入式本地运行。

## 示例

```bash
moltbot agent --to +15555550123 --message "状态更新"
moltbot agent --agent ops --message "总结日志"
moltbot agent --session-id 1234 --message "总结收件箱" --thinking medium
moltbot agent --to +15555550123 --message "追踪日志" --verbose on --json
moltbot agent --to +15555550123 --message "召唤回复" --deliver
moltbot agent --agent ops --message "生成报告" --deliver --reply-channel slack --reply-to "#reports"
```

## 标志

- `--local`：本地运行（需要在 shell 中配置模型提供商 API 密钥）
- `--deliver`：将回复发送到所选频道
- `--channel`：投递频道（`whatsapp|telegram|discord|googlechat|slack|signal|imessage`，默认：`whatsapp`）
- `--reply-to`：覆盖投递目标
- `--reply-channel`：覆盖投递频道
- `--reply-account`：覆盖投递账户 id
- `--thinking <off|minimal|low|medium|high|xhigh>`：持久化思考级别（仅限 GPT-5.2 + Codex 模型）
- `--verbose <on|full|off>`：持久化详细级别
- `--timeout <seconds>`：覆盖代理超时
- `--json`：输出结构化 JSON
