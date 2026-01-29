---
summary: "Research notes: offline memory 系统 for Clawd workspaces (Markdown source-of-truth + derived 索引)"
read_when: 
  - Designing workspace memory (~/clawd) beyond daily Markdown logs
  - Deciding: standalone CLI vs deep Moltbot integration
  - Adding offline recall + reflection (retain/recall/reflect)
---

# 工作空间 Memory v2 (offline): research notes

Target: Clawd-style workspace (`agents.defaults.workspace`, default `~/clawd`) where “memory” is stored as one Markdown file per day (`memory/YYYY-MM-DD.md`) plus a small set of stable files (e.g. `memory.md`, `SOUL.md`).

This doc proposes an **offline-first** memory architecture that keeps Markdown as the canonical, reviewable source of truth, but adds **structured recall** (search, entity summaries, confidence 更新) via a derived 索引.

## Why change?

The current 设置 (one 文件 per day) is excellent for:
- “append-only” journaling
- human editing
- git-backed durability + auditability
- low-friction capture (“just write it down”)

It’s weak for:
- high-recall retrieval (“what did we decide about X?”, “last time we tried Y?”)
- entity-centric answers (“tell me about Alice / The Castle / warelay”) without rereading many 文件
- opinion/preference stability (and evidence when it changes)
- time constraints (“what was true during Nov 2025?”) and conflict resolution

## Design goals

- **Offline**: works without 网络; can run on laptop/Castle; no cloud 依赖项.
- **Explainable**: retrieved items should be attributable (文件 + location) and separable from inference.
- **Low ceremony**: daily 日志记录 stays Markdown, no heavy 模式 work.
- **Incremental**: v1 is useful with FTS only; semantic/vector and graphs are 可选 upgrades.
- **代理-friendly**: makes “recall within 令牌 budgets” easy (return small bundles of facts).

## North star 模型 (Hindsight × Letta)

Two pieces to blend:

1) **Letta/MemGPT-style control loop**
- keep a small “core” always in 上下文 (persona + 键 用户 facts)
- everything else is out-of-上下文 and retrieved via 工具
- memory writes are explicit 工具 calls (append/replace/insert), persisted, then re-injected next turn

2) **Hindsight-style memory substrate**
- separate what’s observed vs what’s believed vs what’s summarized
- support retain/recall/reflect
- confidence-bearing opinions that can evolve with evidence
- entity-aware retrieval + temporal queries (even without full knowledge graphs)

## Proposed architecture (Markdown source-of-truth + derived 索引)

### Canonical store (git-friendly)

Keep `~/clawd` as canonical human-readable memory.

Suggested 工作空间 layout:

```
~/clawd/
  memory.md                    # small: durable facts + preferences (core-ish)
  memory/
    YYYY-MM-DD.md              # daily log (append; narrative)
  bank/                        # “typed” memory pages (stable, reviewable)
    world.md                   # objective facts about the world
    experience.md              # what the agent did (first-person)
    opinions.md                # subjective prefs/judgments + confidence + evidence pointers
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes:
- **Daily 日志 stays daily 日志**. No need to turn it into JSON.
- The `bank/` 文件 are **curated**, produced by reflection jobs, and can still be edited by hand.
- `memory.md` remains “small + core-ish”: the things you want Clawd to 参见 every 会话.

### Derived store (machine recall)

Add a derived 索引 under the 工作空间 (not necessarily git tracked):

```
~/clawd/.memory/index.sqlite
```

Back it with:
- SQLite 模式 for facts + entity links + opinion 元数据
- SQLite **FTS5** for lexical recall (fast, tiny, offline)
- 可选 embeddings 表 for semantic recall (still offline)

The 索引 is always **rebuildable from Markdown**.

## Retain / Recall / Reflect (operational loop)

### Retain: normalize daily 日志 into “facts”

Hindsight’s 键 insight that matters here: store **narrative, self-contained facts**, not tiny snippets.

Practical rule for `memory/YYYY-MM-DD.md`:
- at end of day (or during), add a `## Retain` section with 2–5 bullets that are:
  - narrative (cross-turn 上下文 preserved)
  - self-contained (standalone makes sense later)
  - tagged with 类型 + entity mentions

示例:

```
## Retain
- W @Peter: Currently in Marrakech (Nov 27–Dec 1, 2025) for Andy’s birthday.
- B @warelay: I fixed the Baileys WS crash by wrapping connection.update handlers in try/catch (see memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefers concise replies (&lt;1500 chars) on WhatsApp; long content goes into files.
```

Minimal parsing:
- Type prefix: `W` (world), `B` (experience/biographical), `O` (opinion), `S` (observation/摘要; usually generated)
- Entities: `@Peter`, `@warelay`, etc (slugs map to `bank/entities/*.md`)
- Opinion confidence: `O(c=0.0..1.0)` 可选

If you don’t want authors to think about it: the reflect job can infer these bullets from the rest of the log, but having an explicit `## Retain` section is the easiest “quality lever”.

### Recall: queries over the derived 索引

Recall should support:
- **lexical**: “find exact terms / names / 命令” (FTS5)
- **entity**: “tell me about X” (entity pages + entity-linked facts)
- **temporal**: “what happened around Nov 27” / “since last week”
- **opinion**: “what does Peter prefer?” (with confidence + evidence)

Return 格式 should be 代理-friendly and cite sources:
- `kind` (`world|experience|opinion|observation`)
- `timestamp` (source day, or extracted time range if present)
- `entities` (`["Peter","warelay"]`)
- `content` (the narrative fact)
- `source` (`memory/2025-11-27.md#L12` etc)

### Reflect: produce stable pages + 更新 beliefs

Reflection is a scheduled job (daily or heartbeat `ultrathink`) that:
- updates `bank/entities/*.md` from recent facts (entity summaries)
- updates `bank/opinions.md` confidence based on reinforcement/contradiction
- optionally proposes edits to `memory.md` (“core-ish” durable facts)

Opinion evolution (simple, explainable):
- each opinion has:
  - statement
  - confidence `c ∈ [0,1]`
  - last_updated
  - evidence links (supporting + contradicting fact IDs)
- when new facts arrive:
  - find candidate opinions by entity overlap + similarity (FTS first, embeddings later)
  - 更新 confidence by small deltas; big jumps require strong contradiction + repeated evidence

## CLI integration: standalone vs deep integration

Recommendation: **deep integration in Moltbot**, but keep a separable core 库.

### Why integrate into Moltbot?
- Moltbot already knows:
  - the workspace path (`agents.defaults.workspace`)
  - the 会话 模型 + heartbeats
  - 日志记录 + 故障排除 patterns
- You want the 代理 itself to call the 工具:
  - `moltbot memory recall "…" --k 25 --since 30d`
  - `moltbot memory reflect --since 7d`

### Why still split a 库?
- keep memory logic testable without Gateway/runtime
- reuse from other contexts (local 脚本, future desktop 应用, etc.)

Shape:
The memory tooling is intended to be a small CLI + 库 layer, but this is exploratory only.

## “S-Collide” / SuCo: when to use it (research)

If “S-Collide” refers to **SuCo (Subspace Collision)**: it’s an ANN retrieval approach that targets strong recall/latency tradeoffs by using learned/structured collisions in subspaces (paper: arXiv 2411.14754, 2024).

Pragmatic take for `~/clawd`:
- **don’t start** with SuCo.
- start with SQLite FTS + (可选) simple embeddings; you’ll get most UX wins immediately.
- consider SuCo/HNSW/ScaNN-类 solutions only once:
  - corpus is big (tens/hundreds of thousands of chunks)
  - brute-force embedding search becomes too slow
  - recall quality is meaningfully bottlenecked by lexical search

Offline-friendly alternatives (in increasing complexity):
- SQLite FTS5 + 元数据 过滤器 (zero ML)
- Embeddings + brute force (works surprisingly far if chunk count is low)
- HNSW 索引 (common, robust; needs a 库 绑定)
- SuCo (research-grade; attractive if there’s a solid 实现 you can embed)

Open question:
- what’s the **best** offline embedding 模型 for “personal assistant memory” on your machines (laptop + desktop)?
  - if you already have Ollama: embed with a local 模型; otherwise ship a small embedding 模型 in the toolchain.

## Smallest useful pilot

If you want a minimal, still-useful 版本:

- Add `bank/` entity pages and a `## Retain` section in daily 日志.
- Use SQLite FTS for recall with citations (路径 + line numbers).
- Add embeddings only if recall quality or scale demands it.

## 参考

- Letta / MemGPT concepts: “core memory blocks” + “archival memory” + 工具-driven self-editing memory.
- Hindsight Technical Report: “retain / recall / reflect”, four-网络 memory, narrative fact extraction, opinion confidence evolution.
- SuCo: arXiv 2411.14754 (2024): “Subspace Collision” approximate nearest neighbor retrieval.
