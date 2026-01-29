---
summary: "How Moltbot builds prompt 上下文 and reports 令牌 用法 + costs"
read_when: 
  - Explaining token usage, costs, or context windows
  - Debugging context growth or compaction behavior
---
# 令牌 use & costs

Moltbot tracks **令牌**, not characters. 令牌 are 模型-specific, but most
OpenAI-style 模型 average ~4 characters per 令牌 for English text.

## How the 系统 prompt is built

Moltbot assembles its own 系统 prompt on every run. It includes:

- 工具 list + short descriptions
- Skills list (only metadata; instructions are loaded on demand with `read`)
- Self-更新 instructions
- Workspace + bootstrap files (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` when new). Large files are truncated by `agents.defaults.bootstrapMaxChars` (默认: 20000).
- Time (UTC + 用户 timezone)
- Reply tags + heartbeat 行为
- Runtime 元数据 (主机/OS/模型/thinking)

参见 the full breakdown in [系统 Prompt](/concepts/系统-prompt).

## What counts in the 上下文 window

Everything the 模型 receives counts toward the 上下文 limit:

- 系统 prompt (all sections listed above)
- Conversation history (用户 + assistant 消息)
- 工具 calls and 工具 结果
- Attachments/transcripts (images, audio, 文件)
- Compaction summaries and pruning artifacts
- 提供商 wrappers or safety headers (not visible, but still counted)

For a practical breakdown (per injected file, tools, skills, and system prompt size), use `/context list` or `/context detail`. 参见 [上下文](/concepts/上下文).

## 如何 参见 current 令牌 用法

Use these in chat:

- `/status` → **emoji‑rich 状态 card** with the 会话 模型, 上下文 用法,
  last 响应 输入/输出 令牌, and **estimated cost** (API 键 only).
- `/usage off|tokens|full` → appends a **per-响应 用法 footer** to every reply.
  - Persists per session (stored as `responseUsage`).
  - OAuth 认证 **hides cost** (令牌 only).
- `/usage cost` → shows a local cost 摘要 from Moltbot 会话 日志.

Other surfaces:

- **TUI/Web TUI:** `/status` + `/usage` are supported.
- **CLI:** `moltbot status --usage` and `moltbot channels list` show
  提供商 quota windows (not per-响应 costs).

## Cost estimation (when shown)

Costs are estimated from your 模型 pricing 配置:

```
models.providers.<provider>.models[].cost
```

These are **USD per 1M tokens** for `input`, `output`, `cacheRead`, and
`cacheWrite`. If pricing is missing, Moltbot shows 令牌 only. OAuth 令牌
never show dollar cost.

## 缓存 TTL and pruning impact

提供商 prompt caching only applies within the 缓存 TTL window. Moltbot can
optionally run **缓存-ttl pruning**: it prunes the 会话 once the 缓存 TTL
has expired, then resets the 缓存 window so subsequent 请求 can re-use the
freshly cached 上下文 instead of re-caching the full history. This keeps 缓存
write costs lower when a 会话 goes idle past the TTL.

Configure it in [Gateway 配置](/Gateway/配置) and 参见 the
行为 details in [会话 pruning](/concepts/会话-pruning).

Heartbeat can keep the 缓存 **warm** across idle gaps. If your 模型 缓存 TTL
is `1h`, setting the heartbeat interval just under that (e.g., `55m`) can avoid
re-caching the full prompt, reducing 缓存 write costs.

For Anthropic API pricing, 缓存 reads are significantly cheaper than 输入
令牌, while 缓存 writes are billed at a higher multiplier. 参见 Anthropic’s
prompt caching pricing for the latest rates and TTL multipliers:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

### 示例: keep 1h 缓存 warm with heartbeat

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

## Tips for reducing 令牌 pressure

- Use `/compact` to summarize long 会话.
- Trim large 工具 outputs in your workflows.
- Keep 技能 descriptions short (技能 list is injected into the prompt).
- Prefer smaller 模型 for verbose, exploratory work.

参见 [技能](/工具/技能) for the exact 技能 list overhead formula.
