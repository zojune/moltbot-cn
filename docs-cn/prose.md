---
summary: "OpenProse: .prose workflows, slash 命令, and 状态 in Moltbot"
read_when: 
  - You want to run or write .prose workflows
  - You want to enable the OpenProse plugin
  - You need to understand state storage
---
# OpenProse

OpenProse is a portable, markdown-first workflow format for orchestrating AI sessions. In Moltbot it ships as a plugin that installs an OpenProse skill pack plus a `/prose` slash command. Programs live in `.prose` 文件 and can spawn multiple sub-代理 with explicit control flow.

Official site: https://www.prose.md

## What it can do

- Multi-代理 research + synthesis with explicit parallelism.
- Repeatable approval-safe workflows (code review, incident triage, content pipelines).
- Reusable `.prose` programs you can run across supported 代理 runtimes.

## 安装 + enable

Bundled 插件 are 已禁用 默认情况下. Enable OpenProse:

```bash
moltbot plugins enable open-prose
```

Restart the Gateway after enabling the 插件.

Dev/local checkout: `moltbot plugins install ./extensions/open-prose`

相关 docs: [插件](/插件), [插件 manifest](/插件/manifest), [技能](/工具/技能).

## Slash 命令

OpenProse registers `/prose` as a 用户-invocable 技能 命令. It routes to the OpenProse VM instructions and uses Moltbot 工具 under the hood.

Common 命令:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Example: a simple `.prose` 文件

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## 文件 locations

OpenProse keeps state under `.prose/` in your 工作空间:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

用户-level persistent 代理 live at:

```
~/.prose/agents/
```

## 状态 modes

OpenProse supports multiple 状态 backends:

- **filesystem** (default): `.prose/runs/...`
- **in-上下文**: transient, for small programs
- **sqlite** (experimental): requires `sqlite3` binary
- **postgres** (experimental): requires `psql` and a 连接 字符串

Notes:
- sqlite/postgres are opt-in and experimental.
- postgres 凭据 flow into subagent 日志; use a dedicated, least-privileged DB.

## Remote programs

`/prose run <handle/slug>` resolves to `https://p.prose.md/<handle>/<slug>`.
Direct URLs are fetched as-is. This uses the `web_fetch` tool (or `exec` for POST).

## Moltbot runtime mapping

OpenProse programs map to Moltbot primitives:

| OpenProse concept | Moltbot 工具 |
| --- | --- |
| Spawn session / Task tool | `sessions_spawn` |
| File read/write | `read` / `write` |
| Web fetch | `web_fetch` |

If your 工具 allowlist blocks these 工具, OpenProse programs will fail. 参见 [技能 配置](/工具/技能-配置).

## 安全 + approvals

Treat `.prose` 文件 like code. Review before running. Use Moltbot 工具 allowlists and approval gates to control side effects.

For deterministic, approval-gated workflows, compare with [Lobster](/工具/lobster).
