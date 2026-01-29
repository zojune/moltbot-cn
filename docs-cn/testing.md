---
summary: "测试 kit: unit/e2e/live suites, Docker runners, and what each 测试 covers"
read_when: 
  - Running tests locally or in CI
  - Adding regressions for model/provider bugs
  - Debugging gateway + agent behavior
---

# 测试

Moltbot has three Vitest suites (unit/integration, e2e, live) and a small set of Docker runners.

This doc is a “how we 测试” 指南:
- What each suite covers (and what it deliberately does *not* cover)
- Which 命令 to run for common workflows (local, pre-push, 调试)
- How live 测试 discover 凭据 and select 模型/提供商
- 如何 add regressions for real-world 模型/提供商 issues

## 快速开始

Most days:
- Full gate (expected before push): `pnpm lint && pnpm build && pnpm test`

When you touch 测试 or want extra confidence:
- Coverage gate: `pnpm test:coverage`
- E2E suite: `pnpm test:e2e`

When 调试 real 提供商/模型 (requires real creds):
- Live suite (models + gateway tool/image probes): `pnpm test:live`

提示: when you only need one failing case, prefer narrowing live 测试 via the allowlist env vars described below.

## 测试 suites (what runs where)

Think of the suites as “increasing realism” (and increasing flakiness/cost):

### Unit / integration (默认)

- Command: `pnpm test`
- Config: `vitest.config.ts`
- Files: `src/**/*.test.ts`
- Scope:
  - Pure unit 测试
  - In-进程 integration 测试 (Gateway 认证, routing, tooling, parsing, 配置)
  - Deterministic regressions for known bugs
- Expectations:
  - Runs in CI
  - No real keys 必需
  - Should be fast and stable

### E2E (Gateway smoke)

- Command: `pnpm test:e2e`
- Config: `vitest.e2e.config.ts`
- Files: `src/**/*.e2e.test.ts`
- Scope:
  - Multi-instance Gateway end-to-end 行为
  - WebSocket/HTTP surfaces, 节点 pairing, and heavier networking
- Expectations:
  - Runs in CI (when 已启用 in the pipeline)
  - No real keys 必需
  - More moving parts than unit 测试 (can be slower)

### Live (real 提供商 + real 模型)

- Command: `pnpm test:live`
- Config: `vitest.live.config.ts`
- Files: `src/**/*.live.test.ts`
- Default: **enabled** by `pnpm test:live` (sets `CLAWDBOT_LIVE_TEST=1`)
- Scope:
  - “Does this 提供商/模型 actually work *today* with real creds?”
  - 捕获 提供商 格式 changes, 工具-calling quirks, 认证 issues, and rate limit 行为
- Expectations:
  - Not CI-stable by design (real 网络, real 提供商 策略, quotas, outages)
  - Costs money / uses rate limits
  - Prefer running narrowed subsets instead of “everything”
  - Live runs will source `~/.profile` to pick up missing API keys
  - Anthropic key rotation: set `CLAWDBOT_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."` (or `CLAWDBOT_LIVE_ANTHROPIC_KEY=sk-...`) or multiple `ANTHROPIC_API_KEY*` vars; 测试 will retry on rate limits

## Which suite should I run?

Use this decision 表:
- Editing logic/tests: run `pnpm test` (and `pnpm test:coverage` if you changed a lot)
- Touching gateway networking / WS protocol / pairing: add `pnpm test:e2e`
- Debugging “my bot is down” / provider-specific failures / tool calling: run a narrowed `pnpm test:live`

## Live: 模型 smoke (配置文件 keys)

Live 测试 are split into two layers so we can isolate failures:
- “Direct 模型” tells us the 提供商/模型 can answer at all with the given 键.
- “Gateway smoke” tells us the full Gateway+代理 pipeline works for that 模型 (会话, history, 工具, 沙箱 策略, etc.).

### Layer 1: Direct 模型 completion (no Gateway)

- Test: `src/agents/models.profiles.live.test.ts`
- Goal:
  - Enumerate discovered 模型
  - Use `getApiKeyForModel` to select 模型 you have creds for
  - Run a small completion per 模型 (and targeted regressions where needed)
- 如何 enable:
  - `pnpm test:live` (or `CLAWDBOT_LIVE_TEST=1` if invoking Vitest directly)
- Set `CLAWDBOT_LIVE_MODELS=modern` (or `all`, alias for modern) to actually run this suite; otherwise it skips to keep `pnpm test:live` focused on Gateway smoke
- 如何 select 模型:
  - `CLAWDBOT_LIVE_MODELS=modern` to run the modern allowlist (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `CLAWDBOT_LIVE_MODELS=all` is an alias for the modern allowlist
  - or `CLAWDBOT_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."` (comma allowlist)
- 如何 select 提供商:
  - `CLAWDBOT_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (comma allowlist)
- Where keys come from:
  - 默认情况下: 配置文件 store and env fallbacks
  - Set `CLAWDBOT_LIVE_REQUIRE_PROFILE_KEYS=1` to enforce **配置文件 store** only
- Why this exists:
  - Separates “提供商 API is broken / 键 is invalid” from “Gateway 代理 pipeline is broken”
  - Contains small, isolated regressions (示例: OpenAI 响应/Codex 响应 reasoning replay + 工具-call flows)

### Layer 2: Gateway + dev 代理 smoke (what “@moltbot” actually does)

- Test: `src/gateway/gateway-models.profiles.live.test.ts`
- Goal:
  - Spin up an in-进程 Gateway
  - Create/patch a `agent:dev:*` 会话 (模型 override per run)
  - Iterate 模型-with-keys and assert:
    - “meaningful” 响应 (no 工具)
    - a real 工具 invocation works (read probe)
    - 可选 extra 工具 probes (exec+read probe)
    - OpenAI regression 路径 (工具-call-only → follow-up) keep working
- Probe details (so you can explain failures quickly):
  - `read` probe: the test writes a nonce file in the workspace and asks the agent to `read` it and echo the nonce back.
  - `exec+read` probe: the test asks the agent to `exec`-write a nonce into a temp file, then `read` it back.
  - image probe: the test attaches a generated PNG (cat + randomized code) and expects the model to return `cat <CODE>`.
  - Implementation reference: `src/gateway/gateway-models.profiles.live.test.ts` and `src/gateway/live-image-probe.ts`.
- 如何 enable:
  - `pnpm test:live` (or `CLAWDBOT_LIVE_TEST=1` if invoking Vitest directly)
- 如何 select 模型:
  - 默认: modern allowlist (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `CLAWDBOT_LIVE_GATEWAY_MODELS=all` is an alias for the modern allowlist
  - Or set `CLAWDBOT_LIVE_GATEWAY_MODELS="provider/model"` (or comma list) to narrow
- 如何 select 提供商 (avoid “OpenRouter everything”):
  - `CLAWDBOT_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (comma allowlist)
- 工具 + image probes are always on in this live 测试:
  - `read` probe + `exec+read` probe (工具 stress)
  - image probe runs when the 模型 advertises image 输入 support
  - Flow (high level):
    - Test generates a tiny PNG with “CAT” + random code (`src/gateway/live-image-probe.ts`)
    - Sends it via `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    - Gateway parses attachments into `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    - Embedded 代理 forwards a multimodal 用户 消息 to the 模型
    - Assertion: reply contains `cat` + the code (OCR tolerance: minor mistakes allowed)

Tip: to see what you can test on your machine (and the exact `provider/model` ids), run:

```bash
moltbot models list
moltbot models list --json
```

## Live: Anthropic 设置-令牌 smoke

- Test: `src/agents/anthropic.setup-token.live.test.ts`
- Goal: verify Claude Code CLI 设置-令牌 (or a pasted 设置-令牌 配置文件) can complete an Anthropic prompt.
- Enable:
  - `pnpm test:live` (or `CLAWDBOT_LIVE_TEST=1` if invoking Vitest directly)
  - `CLAWDBOT_LIVE_SETUP_TOKEN=1`
- 令牌 sources (pick one):
  - Profile: `CLAWDBOT_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  - Raw token: `CLAWDBOT_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
- 模型 override (可选):
  - `CLAWDBOT_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

设置 示例:

```bash
moltbot models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
CLAWDBOT_LIVE_SETUP_TOKEN=1 CLAWDBOT_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## Live: CLI backend smoke (Claude Code CLI or other local CLIs)

- Test: `src/gateway/gateway-cli-backend.live.test.ts`
- Goal: 验证 the Gateway + 代理 pipeline using a local CLI backend, without touching your 默认 配置.
- Enable:
  - `pnpm test:live` (or `CLAWDBOT_LIVE_TEST=1` if invoking Vitest directly)
  - `CLAWDBOT_LIVE_CLI_BACKEND=1`
- Defaults:
  - Model: `claude-cli/claude-sonnet-4-5`
  - Command: `claude`
  - Args: `["-p","--output-format","json","--dangerously-skip-permissions"]`
- Overrides (可选):
  - `CLAWDBOT_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  - `CLAWDBOT_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  - `CLAWDBOT_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  - `CLAWDBOT_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  - `CLAWDBOT_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  - `CLAWDBOT_LIVE_CLI_BACKEND_IMAGE_PROBE=1` to send a real image attachment (路径 are injected into the prompt).
  - `CLAWDBOT_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` to pass image 文件 路径 as CLI args instead of prompt injection.
  - `CLAWDBOT_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (or `"list"`) to control how image args are passed when `IMAGE_ARG` is set.
  - `CLAWDBOT_LIVE_CLI_BACKEND_RESUME_PROBE=1` to send a second turn and 验证 resume flow.
- `CLAWDBOT_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` to keep Claude Code CLI MCP 配置 已启用 (默认 disables MCP 配置 with a temporary empty 文件).

示例:

```bash
CLAWDBOT_LIVE_CLI_BACKEND=1 \
  CLAWDBOT_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Recommended live recipes

Narrow, explicit allowlists are fastest and least flaky:

- Single 模型, direct (no Gateway):
  - `CLAWDBOT_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

- Single 模型, Gateway smoke:
  - `CLAWDBOT_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- 工具 calling across several 提供商:
  - `CLAWDBOT_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Google focus (Gemini API 键 + Antigravity):
  - Gemini (API key): `CLAWDBOT_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  - Antigravity (OAuth): `CLAWDBOT_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notes:
- `google/...` uses the Gemini API (API 键).
- `google-antigravity/...` uses the Antigravity OAuth bridge (Cloud Code Assist-style 代理 端点).
- `google-gemini-cli/...` uses the local Gemini CLI on your machine (separate 认证 + tooling quirks).
- Gemini API vs Gemini CLI:
  - API: Moltbot calls Google’s hosted Gemini API over HTTP (API 键 / 配置文件 认证); this is what most 用户 mean by “Gemini”.
  - CLI: Moltbot shells out to a local `gemini` binary; it has its own 认证 and can behave differently (流式/工具 support/版本 skew).

## Live: 模型 matrix (what we cover)

There is no fixed “CI 模型 list” (live is opt-in), but these are the **recommended** 模型 to cover regularly on a dev machine with keys.

### Modern smoke set (工具 calling + image)

This is the “common 模型” run we expect to keep working:
- OpenAI (non-Codex): `openai/gpt-5.2` (optional: `openai/gpt-5.1`)
- OpenAI Codex: `openai-codex/gpt-5.2` (optional: `openai-codex/gpt-5.2-codex`)
- Anthropic: `anthropic/claude-opus-4-5` (or `anthropic/claude-sonnet-4-5`)
- Google (Gemini API): `google/gemini-3-pro-preview` and `google/gemini-3-flash-preview` (avoid older Gemini 2.x 模型)
- Google (Antigravity): `google-antigravity/claude-opus-4-5-thinking` and `google-antigravity/gemini-3-flash`
- Z.AI (GLM): `zai/glm-4.7`
- MiniMax: `minimax/minimax-m2.1`

Run Gateway smoke with 工具 + image:
`CLAWDBOT_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Baseline: 工具 calling (Read + 可选 Exec)

Pick at least one per 提供商 family:
- OpenAI: `openai/gpt-5.2` (or `openai/gpt-5-mini`)
- Anthropic: `anthropic/claude-opus-4-5` (or `anthropic/claude-sonnet-4-5`)
- Google: `google/gemini-3-flash-preview` (or `google/gemini-3-pro-preview`)
- Z.AI (GLM): `zai/glm-4.7`
- MiniMax: `minimax/minimax-m2.1`

可选 additional coverage (nice to have):
- xAI: `xai/grok-4` (or latest available)
- Mistral: `mistral/`… (pick one “工具” capable 模型 you have 已启用)
- Cerebras: `cerebras/`… (if you have access)
- LM Studio: `lmstudio/`… (local; 工具 calling depends on API mode)

### Vision: image send (attachment → multimodal 消息)

Include at least one image-capable model in `CLAWDBOT_LIVE_GATEWAY_MODELS` (Claude/Gemini/OpenAI vision-capable variants, etc.) to exercise the image probe.

### Aggregators / alternate gateways

If you have keys 已启用, we also support 测试 via:
- OpenRouter: `openrouter/...` (hundreds of models; use `moltbot models scan` to find 工具+image capable candidates)
- OpenCode Zen: `opencode/...` (auth via `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

More 提供商 you can include in the live matrix (if you have creds/配置):
- Built-in: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
- Via `models.providers` (custom endpoints): `minimax` (cloud/API), plus any OpenAI/Anthropic-compatible proxy (LM Studio, vLLM, LiteLLM, etc.)

Tip: don’t try to hardcode “all models” in docs. The authoritative list is whatever `discoverModels(...)` returns on your machine + whatever keys are available.

## 凭据 (never commit)

Live 测试 discover 凭据 the same way the CLI does. Practical implications:
- If the CLI works, live 测试 should find the same keys.
- If a live test says “no creds”, debug the same way you’d debug `moltbot models list` / 模型 selection.

- Profile store: `~/.clawdbot/credentials/` (preferred; what “配置文件 keys” means in the 测试)
- Config: `~/.clawdbot/moltbot.json` (or `CLAWDBOT_CONFIG_PATH`)

If you want to rely on env keys (e.g. exported in your `~/.profile`), run local tests after `source ~/.profile`, or use the Docker runners below (they can mount `~/.profile` into the container).

## Deepgram live (audio transcription)

- Test: `src/media-understanding/providers/deepgram/audio.live.test.ts`
- Enable: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## Docker runners (可选 “works in Linux” checks)

These run `pnpm test:live` inside the repo Docker image, mounting your local config dir and workspace (and sourcing `~/.profile` if mounted):

- Direct models: `pnpm test:docker:live-models` (script: `scripts/test-live-models-docker.sh`)
- Gateway + dev agent: `pnpm test:docker:live-gateway` (script: `scripts/test-live-gateway-models-docker.sh`)
- Onboarding wizard (TTY, full scaffolding): `pnpm test:docker:onboard` (script: `scripts/e2e/onboard-docker.sh`)
- Gateway networking (two containers, WS auth + health): `pnpm test:docker:gateway-network` (script: `scripts/e2e/gateway-network-docker.sh`)
- Plugins (custom extension load + registry smoke): `pnpm test:docker:plugins` (script: `scripts/e2e/plugins-docker.sh`)

Useful env vars:

- `CLAWDBOT_CONFIG_DIR=...` (default: `~/.clawdbot`) mounted to `/home/node/.clawdbot`
- `CLAWDBOT_WORKSPACE_DIR=...` (default: `~/clawd`) mounted to `/home/node/clawd`
- `CLAWDBOT_PROFILE_FILE=...` (default: `~/.profile`) mounted to `/home/node/.profile` and sourced before running 测试
- `CLAWDBOT_LIVE_GATEWAY_MODELS=...` / `CLAWDBOT_LIVE_MODELS=...` to narrow the run
- `CLAWDBOT_LIVE_REQUIRE_PROFILE_KEYS=1` to ensure creds come from the 配置文件 store (not env)

## Docs sanity

Run docs checks after doc edits: `pnpm docs:list`.

## Offline regression (CI-safe)

These are “real pipeline” regressions without real 提供商:
- Gateway tool calling (mock OpenAI, real gateway + agent loop): `src/gateway/gateway.tool-calling.mock-openai.test.ts`
- Gateway wizard (WS `wizard.start`/`wizard.next`, writes config + auth enforced): `src/gateway/gateway.wizard.e2e.test.ts`

## 代理 reliability evals (技能)

We already have a few CI-safe 测试 that behave like “代理 reliability evals”:
- Mock tool-calling through the real gateway + agent loop (`src/gateway/gateway.tool-calling.mock-openai.test.ts`).
- End-to-end wizard flows that validate session wiring and config effects (`src/gateway/gateway.wizard.e2e.test.ts`).

What’s still missing for 技能 (参见 [技能](/工具/技能)):
- **Decisioning:** when 技能 are listed in the prompt, does the 代理 pick the right 技能 (or avoid irrelevant ones)?
- **Compliance:** does the agent read `SKILL.md` before use and follow 必需 steps/args?
- **Workflow contracts:** multi-turn scenarios that assert 工具 order, 会话 history carryover, and 沙箱 boundaries.

Future evals should stay deterministic first:
- A scenario runner using mock 提供商 to assert 工具 calls + order, 技能 文件 reads, and 会话 wiring.
- A small suite of 技能-focused scenarios (use vs avoid, gating, prompt injection).
- 可选 live evals (opt-in, env-gated) only after the CI-safe suite is in place.

## Adding regressions (guidance)

When you fix a 提供商/模型 issue discovered in live:
- Add a CI-safe regression if possible (mock/stub 提供商, or capture the exact 请求-shape transformation)
- If it’s inherently live-only (rate limits, 认证 策略), keep the live 测试 narrow and opt-in via env vars
- Prefer targeting the smallest layer that catches the bug:
  - 提供商 请求 conversion/replay bug → direct 模型 测试
  - Gateway 会话/history/工具 pipeline bug → Gateway live smoke or CI-safe Gateway mock 测试
