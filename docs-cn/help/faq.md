---
summary: "Frequently asked questions about Moltbot 设置, 配置, and 用法"
---
# FAQ

Quick answers plus deeper 故障排除 for real-world setups (local dev, VPS, multi-代理, OAuth/API keys, 模型 failover). For runtime diagnostics, 参见 [故障排除](/Gateway/故障排除). For the full 配置 参考, 参见 [配置](/Gateway/配置).

## 表 of contents

- [快速开始 and first-run 设置](#quick-start-and-firstrun-设置)
  - [Im stuck whats the fastest way to get unstuck?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [What’s the recommended way to 安装 and set up Moltbot?](#whats-the-recommended-way-to-安装-and-set-up-moltbot)
  - [How do I open the dashboard after onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  - [How do I authenticate the dashboard (令牌) on localhost vs remote?](#how-do-i-authenticate-the-dashboard-令牌-on-localhost-vs-remote)
  - [What runtime do I need?](#what-runtime-do-i-need)
  - [Does it run on Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [Any tips for Raspberry Pi installs?](#any-tips-for-raspberry-pi-installs)
  - [It is stuck on "wake up my friend" / onboarding will not hatch. What now?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Can I migrate my 设置 to a new machine (Mac mini) without redoing onboarding?](#can-i-migrate-my-设置-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Where do I 参见 what’s new in the latest 版本?](#where-do-i-参见-whats-new-in-the-latest-版本)
  - [I can't access docs.molt.bot (SSL 错误). What now?](#i-cant-access-docsmoltbot-ssl-错误-what-now)
  - [What’s the difference between stable and beta?](#whats-the-difference-between-stable-and-beta)
- [How do I 安装 the beta 版本, and what’s the difference between beta and dev?](#how-do-i-安装-the-beta-版本-and-whats-the-difference-between-beta-and-dev)
  - [How do I 尝试 the latest bits?](#how-do-i-尝试-the-latest-bits)
  - [How long does 安装 and onboarding usually take?](#how-long-does-安装-and-onboarding-usually-take)
  - [Installer stuck? How do I get more feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows 安装 says git not found or moltbot not recognized](#windows-安装-says-git-not-found-or-moltbot-not-recognized)
  - [The docs didn’t answer my question - how do I get a better answer?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [How do I 安装 Moltbot on Linux?](#how-do-i-安装-moltbot-on-linux)
  - [How do I 安装 Moltbot on a VPS?](#how-do-i-安装-moltbot-on-a-vps)
  - [Where are the cloud/VPS 安装 指南?](#where-are-the-cloudvps-安装-指南)
  - [Can I ask Clawd to 更新 itself?](#can-i-ask-clawd-to-更新-itself)
  - [What does the onboarding wizard actually do?](#what-does-the-onboarding-wizard-actually-do)
  - [Do I need a Claude or OpenAI subscription to run this?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Can I use Claude Max subscription without an API 键](#can-i-use-claude-max-subscription-without-an-API-键)
  - [How does Anthropic "设置-令牌" 认证 work?](#how-does-anthropic-setuptoken-认证-work)
  - [Where do I find an Anthropic 设置-令牌?](#where-do-i-find-an-anthropic-setuptoken)
  - [Do you support Claude subscription 认证 (Claude Code OAuth)?](#do-you-support-claude-subscription-认证-claude-code-oauth)
  - [Why am I seeing `HTTP 429: rate_limit_error` from Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Is AWS Bedrock supported?](#is-aws-bedrock-supported)
  - [How does Codex 认证 work?](#how-does-codex-认证-work)
  - [Do you support OpenAI subscription 认证 (Codex OAuth)?](#do-you-support-openai-subscription-认证-codex-oauth)
  - [How do I set up Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  - [Is a local 模型 OK for casual chats?](#is-a-local-模型-ok-for-casual-chats)
  - [How do I keep hosted 模型 traffic in a specific region?](#how-do-i-keep-hosted-模型-traffic-in-a-specific-region)
  - [Do I have to buy a Mac Mini to 安装 this?](#do-i-have-to-buy-a-mac-mini-to-安装-this)
  - [Do I need a Mac mini for iMessage support?](#do-i-need-a-mac-mini-for-imessage-support)
  - [If I buy a Mac mini to run Moltbot, can I connect it to my MacBook Pro?](#if-i-buy-a-mac-mini-to-run-moltbot-can-i-connect-it-to-my-macbook-pro)
  - [Can I use Bun?](#can-i-use-bun)
  - [Telegram: what goes in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Can multiple people use one WhatsApp 数字 with different Moltbots?](#can-multiple-people-use-one-whatsapp-数字-with-different-moltbots)
  - [Can I run a "fast chat" 代理 and an "Opus for coding" 代理?](#can-i-run-a-fast-chat-代理-and-an-opus-for-coding-代理)
  - [Does Homebrew work on Linux?](#does-homebrew-work-on-linux)
  - [What’s the difference between the hackable (git) 安装 and npm 安装?](#whats-the-difference-between-the-hackable-git-安装-and-npm-安装)
  - [Can I switch between npm and git installs later?](#can-i-switch-between-npm-and-git-installs-later)
  - [Should I run the Gateway on my laptop or a VPS?](#should-i-run-the-Gateway-on-my-laptop-or-a-vps)
  - [How 重要 is it to run Moltbot on a dedicated machine?](#how-重要-is-it-to-run-moltbot-on-a-dedicated-machine)
  - [What are the minimum VPS 要求 and recommended OS?](#what-are-the-minimum-vps-要求-and-recommended-os)
  - [Can I run Moltbot in a VM and what are the 要求](#can-i-run-moltbot-in-a-vm-and-what-are-the-要求)
- [What is Moltbot?](#what-is-moltbot)
  - [What is Moltbot, in one paragraph?](#what-is-moltbot-in-one-paragraph)
  - [What’s the 值 proposition?](#whats-the-值-proposition)
  - [I just set it up what should I do first](#i-just-set-it-up-what-should-i-do-first)
  - [What are the top five everyday 用例 for Moltbot](#what-are-the-top-five-everyday-use-cases-for-moltbot)
  - [Can Moltbot help with lead gen outreach ads and blogs for a SaaS](#can-moltbot-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [What are the advantages vs Claude Code for Web 开发?](#what-are-the-advantages-vs-claude-code-for-Web-开发)
- [技能 and automation](#技能-and-automation)
  - [How do I customize 技能 without keeping the repo dirty?](#how-do-i-customize-技能-without-keeping-the-repo-dirty)
  - [Can I load 技能 from a custom folder?](#can-i-load-技能-from-a-custom-folder)
  - [How can I use different 模型 for different tasks?](#how-can-i-use-different-模型-for-different-tasks)
  - [The bot freezes while doing heavy work. How do I offload that?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron or reminders do not fire. What should I check?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [How do I 安装 技能 on Linux?](#how-do-i-安装-技能-on-linux)
  - [Can Moltbot run tasks on a schedule or continuously in the background?](#can-moltbot-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Can I run Apple/macOS-only 技能 from Linux?](#can-i-run-applemacosonly-技能-from-linux)
  - [Do you have a Notion or HeyGen integration?](#do-you-have-a-notion-or-heygen-integration)
  - [How do I 安装 the Chrome extension for browser takeover?](#how-do-i-安装-the-chrome-extension-for-browser-takeover)
- [Sandboxing and memory](#sandboxing-and-memory)
  - [Is there a dedicated sandboxing doc?](#is-there-a-dedicated-sandboxing-doc)
  - [How do I bind a 主机 folder into the 沙箱?](#how-do-i-bind-a-主机-folder-into-the-沙箱)
  - [How does memory work?](#how-does-memory-work)
  - [Memory keeps forgetting things. How do I make it stick?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Does memory persist forever? What are the limits?](#does-memory-persist-forever-what-are-the-limits)
  - [Does semantic memory search require an OpenAI API 键?](#does-semantic-memory-search-require-an-openai-API-键)
- [Where things live on disk](#where-things-live-on-disk)
  - [Is all 数据 used with Moltbot saved locally?](#is-all-数据-used-with-moltbot-saved-locally)
  - [Where does Moltbot store its 数据?](#where-does-moltbot-store-its-数据)
  - [Where should 代理.md / SOUL.md / 用户.md / MEMORY.md live?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [What’s the recommended backup strategy?](#whats-the-recommended-backup-strategy)
  - [How do I completely uninstall Moltbot?](#how-do-i-completely-uninstall-moltbot)
  - [Can 代理 work outside the 工作空间?](#can-代理-work-outside-the-工作空间)
  - [I’m in remote mode - where is the 会话 store?](#im-in-remote-mode-where-is-the-会话-store)
- [配置 basics](#配置-basics)
  - [What 格式 is the 配置? Where is it?](#what-格式-is-the-配置-where-is-it)
  - [I set `gateway.bind: "lan"` (or `"tailnet"`) and now nothing listens / the UI says unauthorized](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Why do I need a 令牌 on localhost now?](#why-do-i-need-a-令牌-on-localhost-now)
  - [Do I have to restart after changing 配置?](#do-i-have-to-restart-after-changing-配置)
  - [How do I enable Web search (and Web fetch)?](#how-do-i-enable-Web-search-and-Web-fetch)
  - [配置.apply wiped my 配置. How do I recover and avoid this?](#configapply-wiped-my-配置-how-do-i-recover-and-avoid-this)
  - [How do I run a central Gateway with specialized workers across devices?](#how-do-i-run-a-central-Gateway-with-specialized-workers-across-devices)
  - [Can the Moltbot browser run headless?](#can-the-moltbot-browser-run-headless)
  - [How do I use Brave for browser control?](#how-do-i-use-brave-for-browser-control)
- [Remote gateways + 节点](#remote-gateways-节点)
  - [How do 命令 propagate between Telegram, the Gateway, and 节点?](#how-do-命令-propagate-between-telegram-the-Gateway-and-节点)
  - [How can my 代理 access my computer if the Gateway is hosted remotely?](#how-can-my-代理-access-my-computer-if-the-Gateway-is-hosted-remotely)
  - [Tailscale is connected but I get no replies. What now?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [Can two Moltbots talk to each other (local + VPS)?](#can-two-moltbots-talk-to-each-other-local-vps)
  - [Do I need separate VPSes for multiple 代理](#do-i-need-separate-vpses-for-multiple-代理)
  - [Is there a benefit to using a 节点 on my personal laptop instead of SSH from a VPS?](#is-there-a-benefit-to-using-a-节点-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Do 节点 run a Gateway 服务?](#do-节点-run-a-Gateway-服务)
  - [Is there an API / RPC way to apply 配置?](#is-there-an-API-rpc-way-to-apply-配置)
  - [What’s a minimal “sane” 配置 for a first 安装?](#whats-a-minimal-sane-配置-for-a-first-安装)
  - [How do I set up Tailscale on a VPS and connect from my Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [How do I connect a Mac 节点 to a remote Gateway (Tailscale Serve)?](#how-do-i-connect-a-mac-节点-to-a-remote-Gateway-tailscale-serve)
  - [Should I 安装 on a second laptop or just add a 节点?](#should-i-安装-on-a-second-laptop-or-just-add-a-节点)
- [Env vars and .env loading](#env-vars-and-env-loading)
  - [How does Moltbot load 环境 变量?](#how-does-moltbot-load-环境-变量)
  - [“I started the Gateway via the 服务 and my env vars disappeared.” What now?](#i-started-the-Gateway-via-the-服务-and-my-env-vars-disappeared-what-now)
  - [I set `COPILOT_GITHUB_TOKEN`, but 模型 状态 shows “Shell env: off.” Why?](#i-set-copilotgithubtoken-but-模型-状态-shows-shell-env-off-why)
- [会话 & multiple chats](#会话-multiple-chats)
  - [How do I start a fresh conversation?](#how-do-i-start-a-fresh-conversation)
  - [Do sessions reset automatically if I never send `/new`?](#do-会话-reset-automatically-if-i-never-send-new)
  - [Is there a way to make a team of Moltbots one CEO and many 代理](#is-there-a-way-to-make-a-team-of-moltbots-one-ceo-and-many-代理)
  - [Why did 上下文 get truncated mid-task? How do I prevent it?](#why-did-上下文-get-truncated-midtask-how-do-i-prevent-it)
  - [How do I completely reset Moltbot but keep it installed?](#how-do-i-completely-reset-moltbot-but-keep-it-installed)
  - [I’m getting “上下文 too large” 错误 - how do I reset or compact?](#im-getting-上下文-too-large-错误-how-do-i-reset-or-compact)
  - [Why am I seeing “LLM 请求 rejected: 消息.N.content.X.tool_use.输入: 字段 必需”?](#why-am-i-seeing-llm-请求-rejected-messagesncontentxtooluseinput-字段-必需)
  - [Why am I getting heartbeat 消息 every 30 minutes?](#why-am-i-getting-heartbeat-消息-every-30-minutes)
  - [Do I need to add a “bot account” to a WhatsApp group?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [How do I get the JID of a WhatsApp group?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Why doesn’t Moltbot reply in a group?](#why-doesnt-moltbot-reply-in-a-group)
  - [Do groups/threads share 上下文 with DMs?](#do-groupsthreads-share-上下文-with-dms)
  - [How many workspaces and 代理 can I create?](#how-many-workspaces-and-代理-can-i-create)
  - [Can I run multiple bots or chats at the same time (Slack), and how should I set that up?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [模型: defaults, selection, aliases, switching](#模型-defaults-selection-aliases-switching)
  - [What is the “默认 模型”?](#what-is-the-默认-模型)
  - [What 模型 do you recommend?](#what-模型-do-you-recommend)
  - [How do I switch 模型 without wiping my 配置?](#how-do-i-switch-模型-without-wiping-my-配置)
  - [Can I use self-hosted 模型 (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-模型-llamacpp-vllm-ollama)
  - [What do Clawd, Flawd, and Krill use for 模型?](#what-do-clawd-flawd-and-krill-use-for-模型)
  - [How do I switch 模型 on the fly (without restarting)?](#how-do-i-switch-模型-on-the-fly-without-restarting)
  - [Can I use GPT 5.2 for daily tasks and Codex 5.2 for coding](#can-i-use-gpt-52-for-daily-tasks-and-codex-52-for-coding)
  - [Why do I 参见 “模型 … is not allowed” and then no reply?](#why-do-i-参见-模型-is-not-allowed-and-then-no-reply)
  - [Why do I 参见 “Unknown 模型: minimax/MiniMax-M2.1”?](#why-do-i-参见-unknown-模型-minimaxminimaxm21)
  - [Can I use MiniMax as my 默认 and OpenAI for complex tasks?](#can-i-use-minimax-as-my-默认-and-openai-for-complex-tasks)
  - [Are opus / sonnet / gpt built‑in shortcuts?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [How do I define/override 模型 shortcuts (aliases)?](#how-do-i-defineoverride-模型-shortcuts-aliases)
  - [How do I add 模型 from other 提供商 like OpenRouter or Z.AI?](#how-do-i-add-模型-from-other-提供商-like-openrouter-or-zai)
- [模型 failover and “All 模型 failed”](#模型-failover-and-all-模型-failed)
  - [How does failover work?](#how-does-failover-work)
  - [What does this 错误 mean?](#what-does-this-错误-mean)
  - [Fix checklist for `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-凭据-found-for-配置文件-anthropicdefault)
  - [Why did it also 尝试 Google Gemini and fail?](#why-did-it-also-尝试-google-gemini-and-fail)
- [认证 配置文件: what they are and 如何 manage them](#认证-配置文件-what-they-are-and-how-to-manage-them)
  - [What is an 认证 配置文件?](#what-is-an-认证-配置文件)
  - [What are typical 配置文件 IDs?](#what-are-typical-配置文件-ids)
  - [Can I control which 认证 配置文件 is tried first?](#can-i-control-which-认证-配置文件-is-tried-first)
  - [OAuth vs API 键: what’s the difference?](#oauth-vs-API-键-whats-the-difference)
- [Gateway: 端口, “already running”, and remote mode](#Gateway-端口-already-running-and-remote-mode)
  - [What 端口 does the Gateway use?](#what-端口-does-the-Gateway-use)
  - [Why does `moltbot gateway status` say `Runtime: running` but `RPC probe: failed`?](#why-does-moltbot-Gateway-状态-say-runtime-running-but-rpc-probe-failed)
  - [Why does `moltbot gateway status` show `Config (cli)` and `Config (service)` different?](#why-does-moltbot-Gateway-状态-show-配置-cli-and-配置-服务-different)
  - [What does “another Gateway instance is already listening” mean?](#what-does-another-Gateway-instance-is-already-listening-mean)
  - [How do I run Moltbot in remote mode (客户端 connects to a Gateway elsewhere)?](#how-do-i-run-moltbot-in-remote-mode-客户端-connects-to-a-Gateway-elsewhere)
  - [The Control UI says “unauthorized” (or keeps reconnecting). What now?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [I set `gateway.bind: "tailnet"` but it can’t bind / nothing listens](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Can I run multiple Gateways on the same 主机?](#can-i-run-multiple-gateways-on-the-same-主机)
  - [What does “invalid handshake” / code 1008 mean?](#what-does-invalid-handshake-code-1008-mean)
- [日志记录 and 调试](#日志记录-and-调试)
  - [Where are 日志?](#where-are-日志)
  - [How do I start/stop/restart the Gateway 服务?](#how-do-i-startstoprestart-the-Gateway-服务)
  - [I closed my terminal on Windows - how do I restart Moltbot?](#i-closed-my-terminal-on-windows-how-do-i-restart-moltbot)
  - [The Gateway is up but replies never arrive. What should I check?](#the-Gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Disconnected from Gateway: no reason" - what now?](#disconnected-from-Gateway-no-reason-what-now)
  - [Telegram setMyCommands fails with 网络 错误. What should I check?](#telegram-setmycommands-fails-with-网络-错误-what-should-i-check)
  - [TUI shows no 输出. What should I check?](#tui-shows-no-输出-what-should-i-check)
  - [How do I completely stop then start the Gateway?](#how-do-i-completely-stop-then-start-the-Gateway)
  - [ELI5: `moltbot gateway restart` vs `moltbot gateway`](#eli5-moltbot-Gateway-restart-vs-moltbot-Gateway)
  - [What’s the fastest way to get more details when something fails?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Media & attachments](#media-attachments)
  - [My 技能 generated an image/PDF, but nothing was sent](#my-技能-generated-an-imagepdf-but-nothing-was-sent)
- [安全 and access control](#安全-and-access-control)
  - [Is it safe to expose Moltbot to inbound DMs?](#is-it-safe-to-expose-moltbot-to-inbound-dms)
  - [Is prompt injection only a concern for public bots?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Should my bot have its own email GitHub account or phone 数字](#should-my-bot-have-its-own-email-github-account-or-phone-数字)
  - [Can I give it autonomy over my text 消息 and is that safe](#can-i-give-it-autonomy-over-my-text-消息-and-is-that-safe)
  - [Can I use cheaper 模型 for personal assistant tasks?](#can-i-use-cheaper-模型-for-personal-assistant-tasks)
  - [I ran `/start` in Telegram but didn’t get a pairing code](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: will it 消息 my contacts? How does pairing work?](#whatsapp-will-it-消息-my-contacts-how-does-pairing-work)
- [Chat 命令, aborting tasks, and “it won’t stop”](#chat-命令-aborting-tasks-and-it-wont-stop)
  - [How do I stop internal 系统 消息 from showing in chat](#how-do-i-stop-internal-系统-消息-from-showing-in-chat)
  - [How do I stop/cancel a running task?](#how-do-i-stopcancel-a-running-task)
  - [How do I send a Discord 消息 from Telegram? (“Cross-上下文 messaging denied”)](#how-do-i-send-a-discord-消息-from-telegram-crosscontext-messaging-denied)
  - [Why does it feel like the bot “ignores” rapid‑fire 消息?](#why-does-it-feel-like-the-bot-ignores-rapidfire-消息)

## First 60 seconds if something's broken

1) **Quick 状态 (first check)**
   ```bash
   moltbot status
   ```
   Fast local 摘要: OS + 更新, Gateway/服务 reachability, 代理/会话, 提供商 配置 + runtime issues (when Gateway is reachable).

2) **Pasteable report (safe to share)**
   ```bash
   moltbot status --all
   ```
   Read-only diagnosis with 日志 tail (令牌 redacted).

3) **Daemon + 端口 状态**
   ```bash
   moltbot gateway status
   ```
   Shows supervisor runtime vs RPC reachability, the probe target URL, and which 配置 the 服务 likely used.

4) **Deep probes**
   ```bash
   moltbot status --deep
   ```
   Runs Gateway health checks + 提供商 probes (requires a reachable Gateway). 参见 [Health](/Gateway/health).

5) **Tail the latest 日志**
   ```bash
   moltbot logs --follow
   ```
   If RPC is down, fall back to:
   ```bash
   tail -f "$(ls -t /tmp/moltbot/moltbot-*.log | head -1)"
   ```
   文件 日志 are separate from 服务 日志; 参见 [日志记录](/日志记录) and [故障排除](/Gateway/故障排除).

6) **Run the doctor (repairs)**
   ```bash
   moltbot doctor
   ```
   Repairs/migrates 配置/状态 + runs health checks. 参见 [Doctor](/Gateway/doctor).

7) **Gateway snapshot**
   ```bash
   moltbot health --json
   moltbot health --verbose   # shows the target URL + config path on errors
   ```
   Asks the running Gateway for a full snapshot (WS-only). 参见 [Health](/Gateway/health).

## 快速开始 and first-run 设置

### Im stuck whats the fastest way to get unstuck

Use a local AI 代理 that can **参见 your machine**. That is far more effective than asking
in Discord, because most "I'm stuck" cases are **local 配置 or 环境 issues** that
remote helpers cannot inspect.

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

These 工具 can read the repo, run 命令, inspect 日志, and help fix your machine-level
设置 (路径, 服务, 权限, 认证 文件). Give them the **full source checkout** via
the hackable (git) 安装:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git
```

This installs Moltbot **from a git checkout**, so the 代理 can read the code + docs and
reason about the exact 版本 you are running. You can always switch back to stable later
by re-running the installer without `--install-method git`.

提示: ask the 代理 to **plan and supervise** the fix (step-by-step), then execute only the
necessary 命令. That keeps changes small and easier to audit.

If you discover a real bug or fix, please 文件 a GitHub issue or send a PR:
https://github.com/moltbot/moltbot/issues
https://github.com/moltbot/moltbot/pulls

Start with these 命令 (share outputs when asking for help):

```bash
moltbot status
moltbot models status
moltbot doctor
```

What they do:
- `moltbot status`: quick snapshot of Gateway/代理 health + basic 配置.
- `moltbot models status`: checks 提供商 认证 + 模型 availability.
- `moltbot doctor`: validates and repairs common 配置/状态 issues.

Other useful CLI checks: `moltbot status --all`, `moltbot logs --follow`,
`moltbot gateway status`, `moltbot health --verbose`.

Quick 调试 loop: [First 60 seconds if something's broken](#first-60-seconds-if-somethings-broken).
安装 docs: [安装](/安装), [Installer flags](/安装/installer), [Updating](/安装/updating).

### Whats the recommended way to 安装 and set up Moltbot

The repo recommends running from source and using the onboarding wizard:

```bash
curl -fsSL https://molt.bot/install.sh | bash
moltbot onboard --install-daemon
```

The wizard can also build UI assets automatically. After onboarding, you typically run the Gateway on 端口 **18789**.

From source (contributors/dev):

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
moltbot onboard
```

If you don’t have a global install yet, run it via `pnpm moltbot onboard`.

### How do I open the dashboard after onboarding

The wizard now opens your browser with a tokenized dashboard URL right after onboarding and also prints the full link (with 令牌) in the 摘要. Keep that tab open; if it didn’t launch, copy/paste the printed URL on the same machine. 令牌 stay local to your 主机-nothing is fetched from the browser.

### How do I authenticate the dashboard 令牌 on localhost vs remote

**Localhost (same machine):**
- Open `http://127.0.0.1:18789/`.
- If it asks for auth, run `moltbot dashboard` and use the tokenized link (`?token=...`).
- The token is the same value as `gateway.auth.token` (or `CLAWDBOT_GATEWAY_TOKEN`) and is stored by the UI after first load.

**Not on localhost:**
- **Tailscale Serve** (recommended): keep bind loopback, run `moltbot gateway --tailscale serve`, open `https://<magicdns>/`. If `gateway.auth.allowTailscale` is `true`, identity headers satisfy 认证 (no 令牌).
- **Tailnet bind**: run `moltbot gateway --bind tailnet --token "<token>"`, open `http://<tailscale-ip>:18789/`, paste 令牌 in dashboard 设置.
- **SSH tunnel**: `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/?token=...` from `moltbot dashboard`.

参见 [Dashboard](/Web/dashboard) and [Web surfaces](/Web) for bind modes and 认证 details.

### What runtime do I need

Node **>= 22** is required. `pnpm` is recommended. Bun is **not recommended** for the Gateway.

### Does it run on Raspberry Pi

Yes. The Gateway is lightweight - docs list **512MB-1GB RAM**, **1 core**, and about **500MB**
disk as enough for personal use, and 注意 that a **Raspberry Pi 4 can run it**.

If you want extra headroom (日志, media, other 服务), **2GB is recommended**, but it’s
not a hard minimum.

提示: a small Pi/VPS can 主机 the Gateway, and you can pair **节点** on your laptop/phone for
local screen/camera/canvas or 命令 execution. 参见 [节点](/节点).

### Any tips for Raspberry Pi installs

Short 版本: it works, but expect rough edges.

- Use a **64-bit** OS and keep 节点 >= 22.
- Prefer the **hackable (git) 安装** so you can 参见 日志 and 更新 fast.
- Start without 渠道/技能, then add them one by one.
- If you hit weird binary issues, it is usually an **ARM compatibility** problem.

Docs: [Linux](/platforms/linux), [安装](/安装).

### It is stuck on wake up my friend onboarding will not hatch What now

That screen depends on the Gateway being reachable and authenticated. The TUI also sends
"Wake up, my friend!" automatically on first hatch. If you 参见 that line with **no reply**
and 令牌 stay at 0, the 代理 never ran.

1) Restart the Gateway:
```bash
moltbot gateway restart
```
2) Check 状态 + 认证:
```bash
moltbot status
moltbot models status
moltbot logs --follow
```
3) If it still hangs, run:
```bash
moltbot doctor
```

If the Gateway is remote, ensure the tunnel/Tailscale 连接 is up and that the UI
is pointed at the right Gateway. 参见 [Remote access](/Gateway/remote).

### Can I migrate my 设置 to a new machine Mac mini without redoing onboarding

Yes. Copy the **状态 目录** and **工作空间**, then run Doctor once. This
keeps your bot “exactly the same” (memory, 会话 history, 认证, and 渠道
状态) as long as you copy **both** locations:

1) 安装 Moltbot on the new machine.
2) Copy `$CLAWDBOT_STATE_DIR` (default: `~/.clawdbot`) from the old machine.
3) Copy your workspace (default: `~/clawd`).
4) Run `moltbot doctor` and restart the Gateway 服务.

That preserves 配置, 认证 配置文件, WhatsApp creds, 会话, and memory. If you’re in
remote mode, remember the Gateway 主机 owns the 会话 store and 工作空间.

**重要:** if you only commit/push your 工作空间 to GitHub, you’re backing
up **memory + bootstrap 文件**, but **not** 会话 history or 认证. Those live
under `~/.clawdbot/` (for example `~/.clawdbot/agents/<agentId>/sessions/`).

相关: [Migrating](/安装/migrating), [Where things live on disk](/help/faq#where-does-moltbot-store-its-数据),
[代理 工作空间](/concepts/代理-工作空间), [Doctor](/Gateway/doctor),
[Remote mode](/Gateway/remote).

### Where do I 参见 whats new in the latest 版本

Check the GitHub changelog:  
https://github.com/moltbot/moltbot/blob/main/CHANGELOG.md

Newest entries are at the top. If the top section is marked **Unreleased**, the next dated
section is the latest shipped 版本. Entries are grouped by **Highlights**, **Changes**, and
**Fixes** (plus docs/other sections when needed).

### I cant access docsmoltbot SSL 错误 What now

Some Comcast/Xfinity connections incorrectly block `docs.molt.bot` via Xfinity
Advanced Security. Disable it or allowlist `docs.molt.bot`, then retry. More
detail: [故障排除](/help/故障排除#docsmoltbot-shows-an-ssl-错误-comcastxfinity).
Please help us unblock it by reporting here: https://spa.xfinity.com/check_url_status.

If you still can't reach the site, the docs are mirrored on GitHub:
https://github.com/moltbot/moltbot/tree/main/docs

### Whats the difference between stable and beta

**Stable** and **beta** are **npm dist‑tags**, not separate code lines:
- `latest` = stable
- `beta` = early build for 测试

We ship builds to **beta**, 测试 them, and once a build is solid we **promote
that same version to `latest`**. That’s why beta and stable can point at the
**same 版本**.

参见 what changed:  
https://github.com/moltbot/moltbot/blob/main/CHANGELOG.md

### How do I 安装 the beta 版本 and whats the difference between beta and dev

**Beta** is the npm dist‑tag `beta` (may match `latest`).  
**Dev** is the moving head of `main` (git); when published, it uses the npm dist‑tag `dev`.

One‑liners (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://molt.bot/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://molt.bot/install.sh | bash -s -- --install-method git
```

Windows installer (PowerShell):
https://molt.bot/安装.ps1

More detail: [开发 渠道](/安装/开发-渠道) and [Installer flags](/安装/installer).

### How long does 安装 and onboarding usually take

Rough 指南:
- **安装:** 2-5 minutes
- **Onboarding:** 5-15 minutes depending on how many 渠道/模型 you configure

If it hangs, use [Installer stuck](/help/faq#installer-stuck-how-do-i-get-more-feedback)
and the fast 调试 loop in [Im stuck](/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

### How do I 尝试 the latest bits

Two 选项:

1) **Dev 渠道 (git checkout):**
```bash
moltbot update --channel dev
```
This switches to the `main` branch and 更新 from source.

2) **Hackable 安装 (from the installer site):**
```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git
```
That gives you a local repo you can edit, then 更新 via git.

If you prefer a clean clone manually, use:
```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm build
```

Docs: [更新](/cli/更新), [开发 渠道](/安装/开发-渠道),
[安装](/安装).

### Installer stuck How do I get more feedback

Re-run the installer with **verbose 输出**:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --verbose
```

Beta 安装 with verbose:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --beta --verbose
```

For a hackable (git) 安装:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git --verbose
```

More 选项: [Installer flags](/安装/installer).

### Windows 安装 says git not found or moltbot not recognized

Two common Windows issues:

**1) npm 错误 spawn git / git not found**
- Install **Git for Windows** and make sure `git` is on your 路径.
- Close and reopen PowerShell, then re-run the installer.

**2) moltbot is not recognized after 安装**
- Your npm global bin folder is not on 路径.
- Check the 路径:
  ```powershell
  npm config get prefix
  ```
- Ensure `<prefix>\\bin` is on PATH (on most systems it is `%AppData%\\npm`).
- Close and reopen PowerShell after updating 路径.

If you want the smoothest Windows 设置, use **WSL2** instead of native Windows.
Docs: [Windows](/platforms/windows).

### The docs didnt answer my question how do I get a better answer

Use the **hackable (git) 安装** so you have the full source and docs locally, then ask
your bot (or Claude/Codex) *from that folder* so it can read the repo and answer precisely.

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git
```

More detail: [安装](/安装) and [Installer flags](/安装/installer).

### How do I 安装 Moltbot on Linux

Short answer: follow the Linux 指南, then run the onboarding wizard.

- Linux quick 路径 + 服务 安装: [Linux](/platforms/linux).
- Full walkthrough: [入门指南](/start/getting-started).
- Installer + 更新: [安装 & 更新](/安装/updating).

### How do I 安装 Moltbot on a VPS

Any Linux VPS works. 安装 on the 服务器, then use SSH/Tailscale to reach the Gateway.

指南: [exe.dev](/platforms/exe-dev), [Hetzner](/platforms/hetzner), [Fly.io](/platforms/fly).  
Remote access: [Gateway remote](/Gateway/remote).

### Where are the cloudVPS 安装 指南

We keep a **hosting hub** with the common 提供商. Pick one and follow the 指南:

- [VPS hosting](/vps) (all 提供商 in one place)
- [Fly.io](/platforms/fly)
- [Hetzner](/platforms/hetzner)
- [exe.dev](/platforms/exe-dev)

工作原理 in the cloud: the **Gateway runs on the 服务器**, and you access it
from your laptop/phone via the Control UI (or Tailscale/SSH). Your 状态 + 工作空间
live on the 服务器, so treat the 主机 as the source of truth and back it up.

You can pair **节点** (Mac/iOS/Android/headless) to that cloud Gateway to access
local screen/camera/canvas or run 命令 on your laptop while keeping the
Gateway in the cloud.

Hub: [Platforms](/platforms). Remote access: [Gateway remote](/Gateway/remote).
节点: [节点](/节点), [节点 CLI](/cli/节点).

### Can I ask Clawd to 更新 itself

Short answer: **possible, not recommended**. The 更新 flow can restart the
Gateway (which drops the active 会话), may need a clean git checkout, and
can prompt for confirmation. Safer: run 更新 from a shell as the operator.

Use the CLI:

```bash
moltbot update
moltbot update status
moltbot update --channel stable|beta|dev
moltbot update --tag <dist-tag|version>
moltbot update --no-restart
```

If you must automate from an 代理:

```bash
moltbot update --yes --no-restart
moltbot gateway restart
```

Docs: [更新](/cli/更新), [Updating](/安装/updating).

### What does the onboarding wizard actually do

`moltbot onboard` is the recommended 设置 路径. In **local mode** it walks you through:

- **模型/认证 设置** (Anthropic **设置-令牌** recommended for Claude subscriptions, OpenAI Codex OAuth supported, API keys 可选, LM Studio local 模型 supported)
- **工作空间** location + bootstrap 文件
- **Gateway 设置** (bind/端口/认证/tailscale)
- **提供商** (WhatsApp, Telegram, Discord, Mattermost (插件), Signal, iMessage)
- **Daemon 安装** (LaunchAgent on macOS; systemd 用户 unit on Linux/WSL2)
- **Health checks** and **技能** selection

It also warns if your configured 模型 is unknown or missing 认证.

### Do I need a Claude or OpenAI subscription to run this

No. You can run Moltbot with **API keys** (Anthropic/OpenAI/others) or with
**local‑only 模型** so your 数据 stays on your 设备. Subscriptions (Claude
Pro/Max or OpenAI Codex) are 可选 ways to authenticate those 提供商.

Docs: [Anthropic](/提供商/anthropic), [OpenAI](/提供商/openai),
[Local 模型](/Gateway/local-模型), [模型](/concepts/模型).

### Can I use Claude Max subscription without an API 键

Yes. You can authenticate with a **设置-令牌**
instead of an API 键. This is the subscription 路径.

Claude Pro/Max subscriptions **do not include an API 键**, so this is the
correct approach for subscription accounts. 重要: you must verify with
Anthropic that this 用法 is allowed under their subscription 策略 and terms.
If you want the most explicit, supported 路径, use an Anthropic API 键.

### How does Anthropic setuptoken 认证 work

`claude setup-token` generates a **token string** via the Claude Code CLI (it is not available in the web console). You can run it on **any machine**. Choose **Anthropic token (paste setup-token)** in the wizard or paste it with `moltbot models auth paste-token --provider anthropic`. The 令牌 is stored as an 认证 配置文件 for the **anthropic** 提供商 and used like an API 键 (no auto-refresh). More detail: [OAuth](/concepts/oauth).

### Where do I find an Anthropic setuptoken

It is **not** in the Anthropic Console. The 设置-令牌 is generated by the **Claude Code CLI** on **any machine**:

```bash
claude setup-token
```

Copy the token it prints, then choose **Anthropic token (paste setup-token)** in the wizard. If you want to run it on the gateway host, use `moltbot models auth setup-token --provider anthropic`. If you ran `claude setup-token` elsewhere, paste it on the gateway host with `moltbot models auth paste-token --provider anthropic`. 参见 [Anthropic](/提供商/anthropic).

### Do you support Claude subscription 认证 (Claude Pro/Max)

Yes — via **设置-令牌**. Moltbot no longer reuses Claude Code CLI OAuth 令牌; use a 设置-令牌 or an Anthropic API 键. Generate the 令牌 anywhere and paste it on the Gateway 主机. 参见 [Anthropic](/提供商/anthropic) and [OAuth](/concepts/oauth).

注意: Claude subscription access is governed by Anthropic’s terms. For 生产 or multi‑用户 workloads, API keys are usually the safer choice.

### Why am I seeing HTTP 429 ratelimiterror from Anthropic

That means your **Anthropic quota/rate limit** is exhausted for the current window. If you
use a **Claude subscription** (设置‑令牌 or Claude Code OAuth), wait for the window to
reset or upgrade your plan. If you use an **Anthropic API 键**, check the Anthropic Console
for 用法/billing and raise limits as needed.

提示: set a **fallback 模型** so Moltbot can keep replying while a 提供商 is rate‑limited.
参见 [模型](/cli/模型) and [OAuth](/concepts/oauth).

### Is AWS Bedrock supported

Yes - via pi‑ai’s **Amazon Bedrock (Converse)** 提供商 with **manual 配置**. You must supply AWS 凭据/region on the Gateway 主机 and add a Bedrock 提供商 entry in your 模型 配置. 参见 [Amazon Bedrock](/bedrock) and [模型 提供商](/提供商/模型). If you prefer a managed 键 flow, an OpenAI‑compatible proxy in front of Bedrock is still a valid 选项.

### How does Codex 认证 work

Moltbot supports **OpenAI Code (Codex)** via OAuth (ChatGPT sign-in). The wizard can run the OAuth flow and will set the default model to `openai-codex/gpt-5.2` when appropriate. 参见 [模型 提供商](/concepts/模型-提供商) and [Wizard](/start/wizard).

### Do you support OpenAI subscription 认证 Codex OAuth

Yes. Moltbot fully supports **OpenAI Code (Codex) subscription OAuth**. The onboarding wizard
can run the OAuth flow for you.

参见 [OAuth](/concepts/oauth), [模型 提供商](/concepts/模型-提供商), and [Wizard](/start/wizard).

### How do I set up Gemini CLI OAuth

Gemini CLI uses a **plugin auth flow**, not a client id or secret in `moltbot.json`.

Steps:
1) Enable the plugin: `moltbot plugins enable google-gemini-cli-auth`
2) Login: `moltbot models auth login --provider google-gemini-cli --set-default`

This stores OAuth 令牌 in 认证 配置文件 on the Gateway 主机. Details: [模型 提供商](/concepts/模型-提供商).

### Is a local 模型 OK for casual chats

Usually no. Moltbot needs large 上下文 + strong safety; small cards truncate and leak. If you must, run the **largest** MiniMax M2.1 build you can locally (LM Studio) and 参见 [/Gateway/local-模型](/Gateway/local-模型). Smaller/quantized 模型 increase prompt-injection risk - 参见 [安全](/Gateway/安全).

### How do I keep hosted 模型 traffic in a specific region

Pick region-pinned endpoints. OpenRouter exposes US-hosted options for MiniMax, Kimi, and GLM; choose the US-hosted variant to keep data in-region. You can still list Anthropic/OpenAI alongside these by using `models.mode: "merge"` so fallbacks stay available while respecting the regioned 提供商 you select.

### Do I have to buy a Mac Mini to 安装 this

No. Moltbot runs on macOS or Linux (Windows via WSL2). A Mac mini is 可选 - some people
buy one as an always‑on 主机, but a small VPS, home 服务器, or Raspberry Pi‑类 box works too.

You only need a Mac **for macOS‑only 工具**. For iMessage, you can keep the Gateway on Linux
and run `imsg` on any Mac over SSH by pointing `channels.imessage.cliPath` at an SSH wrapper.
If you want other macOS‑only 工具, run the Gateway on a Mac or pair a macOS 节点.

Docs: [iMessage](/渠道/imessage), [节点](/节点), [Mac remote mode](/platforms/mac/remote).

### Do I need a Mac mini for iMessage support

You need **some macOS 设备** signed into 消息. It does **not** have to be a Mac mini -
any Mac works. Moltbot’s iMessage integrations run on macOS (BlueBubbles or `imsg`), while
the Gateway can run elsewhere.

Common setups:
- Run the Gateway on Linux/VPS, and point `channels.imessage.cliPath` at an SSH wrapper that
  runs `imsg` on the Mac.
- Run everything on the Mac if you want the simplest single‑machine 设置.

Docs: [iMessage](/渠道/imessage), [BlueBubbles](/渠道/bluebubbles),
[Mac remote mode](/platforms/mac/remote).

### If I buy a Mac mini to run Moltbot can I connect it to my MacBook Pro

Yes. The **Mac mini can run the Gateway**, and your MacBook Pro can connect as a
**节点** (companion 设备). 节点 don’t run the Gateway - they provide extra
capabilities like screen/camera/canvas and `system.run` on that 设备.

Common pattern:
- Gateway on the Mac mini (always‑on).
- MacBook Pro runs the macOS 应用 or a 节点 主机 and pairs to the Gateway.
- Use `moltbot nodes status` / `moltbot nodes list` to 参见 it.

Docs: [节点](/节点), [节点 CLI](/cli/节点).

### Can I use Bun

Bun is **not recommended**. We 参见 runtime bugs, especially with WhatsApp and Telegram.
Use **节点** for stable gateways.

If you still want to experiment with Bun, do it on a non‑生产 Gateway
without WhatsApp/Telegram.

### Telegram what goes in allowFrom

`channels.telegram.allowFrom` is **the human sender’s Telegram user ID** (numeric, recommended) or `@username`. It is not the bot username.

Safer (no third-party bot):
- DM your bot, then run `moltbot logs --follow` and read `from.id`.

Official Bot API:
- DM your bot, then call `https://api.telegram.org/bot<bot_token>/getUpdates` and read `message.from.id`.

Third-party (less private):
- DM `@userinfobot` or `@getidsbot`.

参见 [/渠道/telegram](/渠道/telegram#access-control-dms--groups).

### Can multiple people use one WhatsApp 数字 with different Moltbots

Yes, via **multi‑agent routing**. Bind each sender’s WhatsApp **DM** (peer `kind: "dm"`, sender E.164 like `+15551234567`) to a different `agentId`, so each person gets their own workspace and session store. Replies still come from the **same WhatsApp account**, and DM access control (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) is global per WhatsApp account. 参见 [Multi-代理 Routing](/concepts/multi-代理) and [WhatsApp](/渠道/whatsapp).

### Can I run a fast chat 代理 and an Opus for coding 代理

Yes. Use multi‑代理 routing: give each 代理 its own 默认 模型, then bind inbound routes (提供商 account or specific peers) to each 代理. 示例 配置 lives in [Multi-代理 Routing](/concepts/multi-代理). 另请参阅 [模型](/concepts/模型) and [配置](/Gateway/配置).

### Does Homebrew work on Linux

Yes. Homebrew supports Linux (Linuxbrew). Quick 设置:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

If you run Moltbot via systemd, ensure the service PATH includes `/home/linuxbrew/.linuxbrew/bin` (or your brew prefix) so `brew`-installed 工具 resolve in non‑login shells.
Recent builds also prepend common user bin dirs on Linux systemd services (for example `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) and honor `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR`, and `FNM_DIR` when set.

### Whats the difference between the hackable git 安装 and npm 安装

- **Hackable (git) 安装:** full source checkout, editable, best for contributors.
  You run builds locally and can patch code/docs.
- **npm 安装:** global CLI 安装, no repo, best for “just run it.”
  更新 come from npm dist‑tags.

Docs: [入门指南](/start/getting-started), [Updating](/安装/updating).

### Can I switch between npm and git installs later

Yes. 安装 the other flavor, then run Doctor so the Gateway 服务 points at the new entrypoint.
This **does not delete your 数据** - it only changes the Moltbot code 安装. Your 状态
(`~/.clawdbot`) and workspace (`~/clawd`) stay untouched.

From npm → git:

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm build
moltbot doctor
moltbot gateway restart
```

From git → npm:

```bash
npm install -g moltbot@latest
moltbot doctor
moltbot gateway restart
```

Doctor detects a gateway service entrypoint mismatch and offers to rewrite the service config to match the current install (use `--repair` in automation).

Backup tips: 参见 [Backup strategy](/help/faq#whats-the-recommended-backup-strategy).

### Should I run the Gateway on my laptop or a VPS

Short answer: **if you want 24/7 reliability, use a VPS**. If you want the
lowest friction and you’re okay with sleep/restarts, run it locally.

**Laptop (local Gateway)**
- **Pros:** no 服务器 cost, direct access to local 文件, live browser window.
- **Cons:** sleep/网络 drops = disconnects, OS 更新/reboots interrupt, must stay awake.

**VPS / cloud**
- **Pros:** always‑on, stable 网络, no laptop sleep issues, easier to keep running.
- **Cons:** often run headless (use screenshots), remote 文件 access only, you must SSH for 更新.

**Moltbot-specific 注意:** WhatsApp/Telegram/Slack/Mattermost (插件)/Discord all work fine from a VPS. The only real trade-off is **headless browser** vs a visible window. 参见 [Browser](/工具/browser).

**Recommended 默认:** VPS if you had Gateway disconnects before. Local is great when you’re actively using the Mac and want local 文件 access or UI automation with a visible browser.

### How 重要 is it to run Moltbot on a dedicated machine

Not 必需, but **recommended for reliability and isolation**.

- **Dedicated 主机 (VPS/Mac mini/Pi):** always‑on, fewer sleep/reboot interruptions, cleaner 权限, easier to keep running.
- **Shared laptop/desktop:** totally fine for 测试 and active use, but expect pauses when the machine sleeps or 更新.

If you want the best of both worlds, keep the Gateway on a dedicated 主机 and pair your laptop as a **节点** for local screen/camera/exec 工具. 参见 [节点](/节点).
For 安全 guidance, read [安全](/Gateway/安全).

### What are the minimum VPS 要求 and recommended OS

Moltbot is lightweight. For a basic Gateway + one chat 渠道:

- **Absolute minimum:** 1 vCPU, 1GB RAM, ~500MB disk.
- **Recommended:** 1-2 vCPU, 2GB RAM or more for headroom (日志, media, multiple 渠道). 节点 工具 and browser automation can be resource hungry.

OS: use **Ubuntu LTS** (or any modern Debian/Ubuntu). The Linux 安装 路径 is best tested there.

Docs: [Linux](/platforms/linux), [VPS hosting](/vps).

### Can I run Moltbot in a VM and what are the 要求

Yes. Treat a VM the same as a VPS: it needs to be always on, reachable, and have enough
RAM for the Gateway and any 渠道 you enable.

Baseline guidance:
- **Absolute minimum:** 1 vCPU, 1GB RAM.
- **Recommended:** 2GB RAM or more if you run multiple 渠道, browser automation, or media 工具.
- **OS:** Ubuntu LTS or another modern Debian/Ubuntu.

If you are on Windows, **WSL2 is the easiest VM style 设置** and has the best tooling
compatibility. 参见 [Windows](/platforms/windows), [VPS hosting](/vps).
If you are running macOS in a VM, 参见 [macOS VM](/platforms/macos-vm).

## What is Moltbot?

### What is Moltbot in one paragraph

Moltbot is a personal AI assistant you run on your own devices. It replies on the messaging surfaces you already use (WhatsApp, Telegram, Slack, Mattermost (插件), Discord, Google Chat, Signal, iMessage, WebChat) and can also do voice + a live Canvas on supported platforms. The **Gateway** is the always-on control plane; the assistant is the product.

### Whats the 值 proposition

Moltbot is not “just a Claude wrapper.” It’s a **local-first control plane** that lets you run a
capable assistant on **your own hardware**, reachable from the chat apps you already use, with
stateful 会话, memory, and 工具 - without handing control of your workflows to a hosted
SaaS.

Highlights:
- **Your devices, your 数据:** run the Gateway wherever you want (Mac, Linux, VPS) and keep the
  工作空间 + 会话 history local.  
- **Real 渠道, not a Web 沙箱:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc,
  plus mobile voice and Canvas on supported platforms.  
- **模型-agnostic:** use Anthropic, OpenAI, MiniMax, OpenRouter, etc., with per‑代理 routing
  and failover.  
- **Local-only 选项:** run local 模型 so **all 数据 can stay on your 设备** if you want.
- **Multi-代理 routing:** separate 代理 per 渠道, account, or task, each with its own
  工作空间 and defaults.  
- **Open source and hackable:** inspect, extend, and self-主机 without vendor lock‑in.

Docs: [Gateway](/Gateway), [渠道](/渠道), [Multi‑代理](/concepts/multi-代理),
[Memory](/concepts/memory).

### I just set it up what should I do first

Good first projects:
- Build a website (WordPress, Shopify, or a simple static site).
- Prototype a mobile 应用 (outline, screens, API plan).
- Organize 文件 and folders (cleanup, naming, tagging).
- Connect Gmail and automate summaries or follow ups.

It can handle large tasks, but it works best when you split them into phases and
use sub 代理 for parallel work.

### What are the top five everyday 用例 for Moltbot

Everyday wins usually look like:
- **Personal briefings:** summaries of inbox, calendar, and news you care about.
- **Research and drafting:** quick research, summaries, and first drafts for emails or docs.
- **Reminders and follow ups:** cron or heartbeat driven nudges and checklists.
- **Browser automation:** filling forms, collecting 数据, and repeating Web tasks.
- **Cross 设备 coordination:** send a task from your phone, let the Gateway run it on a 服务器, and get the 结果 back in chat.

### Can Moltbot help with lead gen outreach ads and blogs for a SaaS

Yes for **research, qualification, and drafting**. It can scan sites, build shortlists,
summarize prospects, and write outreach or ad copy drafts.

For **outreach or ad runs**, keep a human in the loop. Avoid spam, follow local laws and
平台 策略, and review anything before it is sent. The safest pattern is to let
Moltbot draft and you approve.

Docs: [安全](/Gateway/安全).

### What are the advantages vs Claude Code for Web 开发

Moltbot is a **personal assistant** and coordination layer, not an IDE replacement. Use
Claude Code or Codex for the fastest direct coding loop inside a repo. Use Moltbot when you
want durable memory, cross-设备 access, and 工具 orchestration.

Advantages:
- **Persistent memory + 工作空间** across 会话
- **Multi-平台 access** (WhatsApp, Telegram, TUI, WebChat)
- **工具 orchestration** (browser, 文件, scheduling, 钩子)
- **Always-on Gateway** (run on a VPS, interact from anywhere)
- **节点** for local browser/screen/camera/exec

Showcase: https://molt.bot/showcase

## 技能 and automation

### How do I customize 技能 without keeping the repo dirty

Use managed overrides instead of editing the repo copy. Put your changes in `~/.clawdbot/skills/<name>/SKILL.md` (or add a folder via `skills.load.extraDirs` in `~/.clawdbot/moltbot.json`). Precedence is `<workspace>/skills` > `~/.clawdbot/skills` > bundled, so managed overrides win without touching git. Only upstream-worthy edits should live in the repo and go out as PRs.

### Can I load 技能 from a custom folder

Yes. Add extra directories via `skills.load.extraDirs` in `~/.clawdbot/moltbot.json` (lowest precedence). Default precedence remains: `<workspace>/skills` → `~/.clawdbot/skills` → bundled → `skills.load.extraDirs`. `clawdhub` installs into `./skills` by default, which Moltbot treats as `<workspace>/skills`.

### How can I use different 模型 for different tasks

Today the supported patterns are:
- **Cron jobs**: isolated jobs can set a `model` override per job.
- **Sub-代理**: route tasks to separate 代理 with different 默认 模型.
- **On-demand switch**: use `/model` to switch the current 会话 模型 at any time.

参见 [Cron jobs](/automation/cron-jobs), [Multi-代理 Routing](/concepts/multi-代理), and [Slash 命令](/工具/slash-命令).

### The bot freezes while doing heavy work How do I offload that

Use **sub-代理** for long or parallel tasks. Sub-代理 run in their own 会话,
return a 摘要, and keep your main chat responsive.

Ask your bot to "spawn a sub-agent for this task" or use `/subagents`.
Use `/status` in chat to 参见 what the Gateway is doing right now (and whether it is busy).

令牌 提示: long tasks and sub-代理 both consume 令牌. If cost is a concern, set a
cheaper model for sub-agents via `agents.defaults.subagents.model`.

Docs: [Sub-代理](/工具/subagents).

### Cron or reminders do not fire What should I check

Cron runs inside the Gateway 进程. If the Gateway is not running continuously,
scheduled jobs will not run.

Checklist:
- Confirm cron is enabled (`cron.enabled`) and `CLAWDBOT_SKIP_CRON` is not set.
- Check the Gateway is running 24/7 (no sleep/restarts).
- Verify timezone settings for the job (`--tz` vs 主机 timezone).

调试:
```bash
moltbot cron run <jobId> --force
moltbot cron runs --id <jobId> --limit 50
```

Docs: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat).

### How do I 安装 技能 on Linux

Use **ClawdHub** (CLI) or drop 技能 into your 工作空间. The macOS 技能 UI isn’t available on Linux.
Browse 技能 at https://clawdhub.com.

安装 the ClawdHub CLI (pick one 包 manager):

```bash
npm i -g clawdhub
```

```bash
pnpm add -g clawdhub
```

### Can Moltbot run tasks on a schedule or continuously in the background

Yes. Use the Gateway scheduler:

- **Cron jobs** for scheduled or recurring tasks (persist across restarts).
- **Heartbeat** for “main 会话” periodic checks.
- **Isolated jobs** for autonomous 代理 that post summaries or deliver to chats.

Docs: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat),
[Heartbeat](/Gateway/heartbeat).

**Can I run Apple macOS only 技能 from Linux**

Not directly. macOS skills are gated by `metadata.moltbot.os` plus required binaries, and skills only appear in the system prompt when they are eligible on the **Gateway host**. On Linux, `darwin`-only skills (like `imsg`, `apple-notes`, `apple-reminders`) will not load unless you override the gating.

You have three supported patterns:

**选项 A - run the Gateway on a Mac (simplest).**  
Run the Gateway where the macOS binaries exist, then connect from Linux in [remote mode](#how-do-i-run-moltbot-in-remote-mode-客户端-connects-to-a-Gateway-elsewhere) or over Tailscale. The 技能 load normally because the Gateway 主机 is macOS.

**选项 B - use a macOS 节点 (no SSH).**  
Run the Gateway on Linux, pair a macOS node (menubar app), and set **Node Run Commands** to "Always Ask" or "Always Allow" on the Mac. Moltbot can treat macOS-only skills as eligible when the required binaries exist on the node. The agent runs those skills via the `nodes` 工具. If you choose "Always Ask", approving "Always Allow" in the prompt adds that 命令 to the allowlist.

**选项 C - proxy macOS binaries over SSH (advanced).**  
Keep the Gateway on Linux, but make the 必需 CLI binaries resolve to SSH wrappers that run on a Mac. Then override the 技能 to allow Linux so it stays eligible.

1) Create an SSH wrapper for the binary (example: `imsg`):
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/imsg "$@"
   ```
2) Put the wrapper on `PATH` on the Linux host (for example `~/bin/imsg`).
3) Override the skill metadata (workspace or `~/.clawdbot/skills`) to allow Linux:
   ```markdown
   ---
   name: imsg
   description: iMessage/SMS CLI for listing chats, history, watch, and sending.
   metadata: {"moltbot":{"os":["darwin","linux"],"requires":{"bins":["imsg"]}}}
   ---
   ```
4) Start a new 会话 so the 技能 snapshot refreshes.

For iMessage specifically, you can also point `channels.imessage.cliPath` at an SSH wrapper (Moltbot only needs stdio). 参见 [iMessage](/渠道/imessage).

### Do you have a Notion or HeyGen integration

Not built‑in today.

选项:
- **Custom 技能 / 插件:** best for reliable API access (Notion/HeyGen both have APIs).
- **Browser automation:** works without code but is slower and more fragile.

If you want to keep 上下文 per 客户端 (agency workflows), a simple pattern is:
- One Notion page per 客户端 (上下文 + preferences + active work).
- Ask the 代理 to fetch that page at the start of a 会话.

If you want a native integration, open a 功能 请求 or build a 技能
targeting those APIs.

安装 技能:

```bash
clawdhub install <skill-slug>
clawdhub update --all
```

ClawdHub installs into `./skills` under your current directory (or falls back to your configured Moltbot workspace); Moltbot treats that as `<workspace>/skills` on the next session. For shared skills across agents, place them in `~/.clawdbot/skills/<name>/SKILL.md`. Some 技能 expect binaries installed via Homebrew; on Linux that means Linuxbrew (参见 the Homebrew Linux FAQ entry above). 参见 [技能](/工具/技能) and [ClawdHub](/工具/clawdhub).

### How do I 安装 the Chrome extension for browser takeover

Use the built-in installer, then load the unpacked extension in Chrome:

```bash
moltbot browser extension install
moltbot browser extension path
```

Then Chrome → `chrome://extensions` → enable “Developer mode” → “Load unpacked” → pick that folder.

Full 指南 (including remote Gateway + 安全 notes): [Chrome extension](/工具/chrome-extension)

If the Gateway runs on the same machine as Chrome (默认 设置), you usually **do not** need anything extra.
If the Gateway runs elsewhere, run a 节点 主机 on the browser machine so the Gateway can proxy browser 操作.
You still need to click the extension button on the tab you want to control (it doesn’t auto-attach).

## Sandboxing and memory

### Is there a dedicated sandboxing doc

Yes. 参见 [Sandboxing](/Gateway/sandboxing). For Docker-specific 设置 (full Gateway in Docker or 沙箱 images), 参见 [Docker](/安装/docker).

**Can I keep DMs personal but make groups public sandboxed with one 代理**

Yes - if your private traffic is **DMs** and your public traffic is **groups**.

Use `agents.defaults.sandbox.mode: "non-main"` so group/channel sessions (non-main keys) run in Docker, while the main DM session stays on-host. Then restrict what tools are available in sandboxed sessions via `tools.sandbox.tools`.

设置 walkthrough + 示例 配置: [Groups: personal DMs + public groups](/concepts/groups#pattern-personal-dms-public-groups-single-代理)

键 配置 参考: [Gateway 配置](/Gateway/配置#agentsdefaultssandbox)

### How do I bind a 主机 folder into the 沙箱

Set `agents.defaults.sandbox.docker.binds` to `["host:path:mode"]` (e.g., `"/home/user/src:/src:ro"`). Global + per-agent binds merge; per-agent binds are ignored when `scope: "shared"`. Use `:ro` for anything sensitive and remember binds bypass the 沙箱 filesystem walls. 参见 [Sandboxing](/Gateway/sandboxing#custom-bind-mounts) and [沙箱 vs 工具 策略 vs Elevated](/Gateway/沙箱-vs-工具-策略-vs-elevated#bind-mounts-安全-quick-check) for 示例 and safety notes.

### How does memory work

Moltbot memory is just Markdown 文件 in the 代理 工作空间:
- Daily notes in `memory/YYYY-MM-DD.md`
- Curated long-term notes in `MEMORY.md` (main/private 会话 only)

Moltbot also runs a **silent pre-compaction memory flush** to remind the 模型
to write durable notes before auto-compaction. This only runs when the 工作空间
is writable (read-only 沙箱 skip it). 参见 [Memory](/concepts/memory).

### Memory keeps forgetting things How do I make it stick

Ask the bot to **write the fact to memory**. Long-term notes belong in `MEMORY.md`,
short-term context goes into `memory/YYYY-MM-DD.md`.

This is still an area we are improving. It helps to remind the 模型 to store memories;
it will know what to do. If it keeps forgetting, verify the Gateway is using the same
工作空间 on every run.

Docs: [Memory](/concepts/memory), [代理 工作空间](/concepts/代理-工作空间).

### Does semantic memory search require an OpenAI API 键

Only if you use **OpenAI embeddings**. Codex OAuth covers chat/completions and
does **not** grant embeddings access, so **signing in with Codex (OAuth or the
Codex CLI login)** does not help for semantic memory search. OpenAI embeddings
still need a real API key (`OPENAI_API_KEY` or `models.providers.openai.apiKey`).

If you don’t set a 提供商 explicitly, Moltbot auto-selects a 提供商 when it
can resolve an API key (auth profiles, `models.providers.*.apiKey`, or env vars).
It prefers OpenAI if an OpenAI 键 resolves, otherwise Gemini if a Gemini 键
resolves. If neither 键 is available, memory search stays 已禁用 until you
configure it. If you have a local 模型 路径 configured and present, Moltbot
prefers `local`.

If you’d rather stay local, set `memorySearch.provider = "local"` (and optionally
`memorySearch.fallback = "none"`). If you want Gemini embeddings, set
`memorySearch.provider = "gemini"` and provide `GEMINI_API_KEY` (or
`memorySearch.remote.apiKey`). We support **OpenAI, Gemini, or local** embedding
模型 - 参见 [Memory](/concepts/memory) for the 设置 details.

### Does memory persist forever What are the limits

Memory 文件 live on disk and persist until you delete them. The limit is your
存储, not the 模型. The **会话 上下文** is still limited by the 模型
上下文 window, so long conversations can compact or truncate. That is why
memory search exists - it pulls only the relevant parts back into 上下文.

Docs: [Memory](/concepts/memory), [上下文](/concepts/上下文).

## Where things live on disk

### Is all 数据 used with Moltbot saved locally

No - **Moltbot’s 状态 is local**, but **external 服务 still 参见 what you send them**.

- **Local 默认情况下:** 会话, memory 文件, 配置, and 工作空间 live on the Gateway 主机
  (`~/.clawdbot` + your 工作空间 目录).
- **Remote by necessity:** 消息 you send to 模型 提供商 (Anthropic/OpenAI/etc.) go to
  their APIs, and chat platforms (WhatsApp/Telegram/Slack/etc.) store 消息 数据 on their
  服务器.
- **You control the footprint:** using local 模型 keeps prompts on your machine, but 渠道
  traffic still goes through the 渠道’s 服务器.

相关: [代理 工作空间](/concepts/代理-工作空间), [Memory](/concepts/memory).

### Where does Moltbot store its 数据

Everything lives under `$CLAWDBOT_STATE_DIR` (default: `~/.clawdbot`):

| 路径 | Purpose |
|------|---------|
| `$CLAWDBOT_STATE_DIR/moltbot.json` | Main 配置 (JSON5) |
| `$CLAWDBOT_STATE_DIR/credentials/oauth.json` | Legacy OAuth import (copied into 认证 配置文件 on first use) |
| `$CLAWDBOT_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | 认证 配置文件 (OAuth + API keys) |
| `$CLAWDBOT_STATE_DIR/agents/<agentId>/agent/auth.json` | Runtime 认证 缓存 (managed automatically) |
| `$CLAWDBOT_STATE_DIR/credentials/` | Provider state (e.g. `whatsapp/<accountId>/creds.json`) |
| `$CLAWDBOT_STATE_DIR/agents/` | Per‑代理 状态 (agentDir + 会话) |
| `$CLAWDBOT_STATE_DIR/agents/<agentId>/sessions/` | Conversation history & 状态 (per 代理) |
| `$CLAWDBOT_STATE_DIR/agents/<agentId>/sessions/sessions.json` | 会话 元数据 (per 代理) |

Legacy single‑agent path: `~/.clawdbot/agent/*` (migrated by `moltbot doctor`).

Your **workspace** (AGENTS.md, memory files, skills, etc.) is separate and configured via `agents.defaults.workspace` (default: `~/clawd`).

### Where should AGENTSmd SOULmd USERmd MEMORYmd live

These files live in the **agent workspace**, not `~/.clawdbot`.

- **Workspace (per agent)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (or `memory.md`), `memory/YYYY-MM-DD.md`, optional `HEARTBEAT.md`.
- **State dir (`~/.clawdbot`)**: 配置, 凭据, 认证 配置文件, 会话, 日志,
  and shared skills (`~/.clawdbot/skills`).

Default workspace is `~/clawd`, configurable via:

```json5
{
  agents: { defaults: { workspace: "~/clawd" } }
}
```

If the bot “forgets” after a restart, confirm the Gateway is using the same
工作空间 on every launch (and remember: remote mode uses the **Gateway 主机’s**
工作空间, not your local laptop).

提示: if you want a durable 行为 or preference, ask the bot to **write it into
代理.md or MEMORY.md** rather than relying on chat history.

参见 [代理 工作空间](/concepts/代理-工作空间) and [Memory](/concepts/memory).

### Whats the recommended backup strategy

Put your **代理 工作空间** in a **private** git repo and back it up somewhere
private (例如 GitHub private). This captures memory + 代理/SOUL/用户
文件, and lets you restore the assistant’s “mind” later.

Do **not** commit anything under `~/.clawdbot` (凭据, 会话, 令牌).
If you need a full restore, back up both the 工作空间 and the 状态 目录
separately (参见 the 迁移 question above).

Docs: [代理 工作空间](/concepts/代理-工作空间).

### How do I completely uninstall Moltbot

参见 the dedicated 指南: [Uninstall](/安装/uninstall).

### Can 代理 work outside the 工作空间

Yes. The 工作空间 is the **默认 cwd** and memory anchor, not a hard 沙箱.
Relative 路径 resolve inside the 工作空间, but absolute 路径 can access other
主机 locations unless sandboxing is 已启用. If you need isolation, use
[`agents.defaults.sandbox`](/Gateway/sandboxing) or per‑代理 沙箱 设置. If you
want a repo to be the 默认 working 目录, point that 代理’s
`workspace` to the repo root. The Moltbot repo is just source code; keep the
工作空间 separate unless you intentionally want the 代理 to work inside it.

示例 (repo as 默认 cwd):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo"
    }
  }
}
```

### Im in remote mode where is the 会话 store

会话 状态 is owned by the **Gateway 主机**. If you’re in remote mode, the 会话 store you care about is on the remote machine, not your local laptop. 参见 [会话 management](/concepts/会话).

## 配置 basics

### What 格式 is the 配置 Where is it

Moltbot reads an optional **JSON5** config from `$CLAWDBOT_CONFIG_PATH` (default: `~/.clawdbot/moltbot.json`):

```
$CLAWDBOT_CONFIG_PATH
```

If the file is missing, it uses safe‑ish defaults (including a default workspace of `~/clawd`).

### I set gatewaybind lan or tailnet and now nothing listens the UI says unauthorized

Non-loopback binds **require auth**. Configure `gateway.auth.mode` + `gateway.auth.token` (or use `CLAWDBOT_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me"
    }
  }
}
```

Notes:
- `gateway.remote.token` is for **remote CLI calls** only; it does not enable local Gateway 认证.
- The Control UI authenticates via `connect.params.auth.token` (stored in 应用/UI 设置). Avoid putting 令牌 in URLs.

### Why do I need a 令牌 on localhost now

The wizard generates a Gateway 令牌 默认情况下 (even on loopback) so **local WS 客户端 must authenticate**. This blocks other local 进程 from calling the Gateway. Paste the 令牌 into the Control UI 设置 (or your 客户端 配置) to connect.

If you **really** want open loopback, remove `gateway.auth` from your config. Doctor can generate a token for you any time: `moltbot doctor --generate-gateway-token`.

### Do I have to restart after changing 配置

The Gateway watches the 配置 and supports hot‑reload:

- `gateway.reload.mode: "hybrid"` (默认): hot‑apply safe changes, restart for critical ones
- `hot`, `restart`, `off` are also supported

### How do I enable Web search and Web fetch

`web_fetch` works without an API key. `web_search` requires a Brave Search API
key. **Recommended:** run `moltbot configure --section web` to store it in
`tools.web.search.apiKey`. Environment alternative: set `BRAVE_API_KEY` for the
Gateway 进程.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5
      },
      fetch: {
        enabled: true
      }
    }
  }
}
```

Notes:
- If you use allowlists, add `web_search`/`web_fetch` or `group:web`.
- `web_fetch` is 已启用 默认情况下 (unless explicitly 已禁用).
- Daemons read env vars from `~/.clawdbot/.env` (or the 服务 环境).

Docs: [Web 工具](/工具/Web).

### How do I run a central Gateway with specialized workers across devices

The common pattern is **one Gateway** (e.g. Raspberry Pi) plus **节点** and **代理**:

- **Gateway (central):** owns 渠道 (Signal/WhatsApp), routing, and 会话.
- **Nodes (devices):** Macs/iOS/Android connect as peripherals and expose local tools (`system.run`, `canvas`, `camera`).
- **代理 (workers):** separate brains/workspaces for special 角色 (e.g. “Hetzner ops”, “Personal 数据”).
- **Sub‑代理:** spawn background work from a main 代理 when you want parallelism.
- **TUI:** connect to the Gateway and switch 代理/会话.

Docs: [节点](/节点), [Remote access](/Gateway/remote), [Multi-代理 Routing](/concepts/multi-代理), [Sub-代理](/工具/subagents), [TUI](/tui).

### Can the Moltbot browser run headless

Yes. It’s a 配置 选项:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } }
    }
  }
}
```

Default is `false` (headful). Headless is more likely to 触发器 anti‑bot checks on some sites. 参见 [Browser](/工具/browser).

Headless uses the **same Chromium engine** and works for most automation (forms, clicks, scraping, logins). The main differences:
- No visible browser window (use screenshots if you need visuals).
- Some sites are stricter about automation in headless mode (CAPTCHAs, anti‑bot).
  例如, X/Twitter often blocks headless 会话.

### How do I use Brave for browser control

Set `browser.executablePath` to your Brave binary (or any Chromium-based browser) and restart the Gateway.
参见 the full 配置 示例 in [Browser](/工具/browser#use-brave-or-another-chromium-based-browser).

## Remote gateways + 节点

### How do 命令 propagate between Telegram the Gateway and 节点

Telegram 消息 are handled by the **Gateway**. The Gateway runs the 代理 and
only then calls 节点 over the **Gateway WebSocket** when a 节点 工具 is needed:

Telegram → Gateway → Agent → `node.*` → 节点 → Gateway → Telegram

节点 don’t 参见 inbound 提供商 traffic; they only receive 节点 RPC calls.

### How can my 代理 access my computer if the Gateway is hosted remotely

Short answer: **pair your computer as a 节点**. The Gateway runs elsewhere, but it can
call `node.*` 工具 (screen, camera, 系统) on your local machine over the Gateway WebSocket.

Typical 设置:
1) Run the Gateway on the always‑on 主机 (VPS/home 服务器).
2) Put the Gateway 主机 + your computer on the same tailnet.
3) Ensure the Gateway WS is reachable (tailnet bind or SSH tunnel).
4) Open the macOS 应用 locally and connect in **Remote over SSH** mode (or direct tailnet)
   so it can register as a 节点.
5) Approve the 节点 on the Gateway:
   ```bash
   moltbot nodes pending
   moltbot nodes approve <requestId>
   ```

No separate TCP bridge is 必需; 节点 connect over the Gateway WebSocket.

Security reminder: pairing a macOS node allows `system.run` on that machine. Only
pair devices you trust, and review [安全](/Gateway/安全).

Docs: [节点](/节点), [Gateway 协议](/Gateway/协议), [macOS remote mode](/platforms/mac/remote), [安全](/Gateway/安全).

### Tailscale is connected but I get no replies What now

Check the basics:
- Gateway is running: `moltbot gateway status`
- Gateway health: `moltbot status`
- Channel health: `moltbot channels status`

Then verify 认证 and routing:
- If you use Tailscale Serve, make sure `gateway.auth.allowTailscale` is set correctly.
- If you connect via SSH tunnel, confirm the local tunnel is up and points at the right 端口.
- Confirm your allowlists (DM or group) include your account.

Docs: [Tailscale](/Gateway/tailscale), [Remote access](/Gateway/remote), [渠道](/渠道).

### Can two Moltbots talk to each other local VPS

Yes. There is no built-in "bot-to-bot" bridge, but you can wire it up in a few
reliable ways:

**Simplest:** use a normal chat 渠道 both bots can access (Telegram/Slack/WhatsApp).
Have Bot A send a 消息 to Bot B, then let Bot B reply as usual.

**CLI bridge (generic):** run a 脚本 that calls the other Gateway with
`moltbot agent --message ... --deliver`, targeting a chat where the other bot
listens. If one bot is on a remote VPS, point your CLI at that remote Gateway
via SSH/Tailscale (参见 [Remote access](/Gateway/remote)).

示例 pattern (run from a machine that can reach the target Gateway):
```bash
moltbot agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

提示: add a guardrail so the two bots do not loop endlessly (mention-only, 渠道
allowlists, or a "do not reply to bot 消息" 规则).

Docs: [Remote access](/Gateway/remote), [代理 CLI](/cli/代理), [代理 send](/工具/代理-send).

### Do I need separate VPSes for multiple 代理

No. One Gateway can 主机 multiple 代理, each with its own 工作空间, 模型 defaults,
and routing. That is the normal 设置 and it is much cheaper and simpler than running
one VPS per 代理.

Use separate VPSes only when you need hard isolation (安全 boundaries) or very
different configs that you do not want to share. Otherwise, keep one Gateway and
use multiple 代理 or sub-代理.

### Is there a benefit to using a 节点 on my personal laptop instead of SSH from a VPS

Yes - 节点 are the first‑类 way to reach your laptop from a remote Gateway, and they
unlock more than shell access. The Gateway runs on macOS/Linux (Windows via WSL2) and is
lightweight (a small VPS or Raspberry Pi-类 box is fine; 4 GB RAM is plenty), so a common
设置 is an always‑on 主机 plus your laptop as a 节点.

- **No inbound SSH 必需.** 节点 connect out to the Gateway WebSocket and use 设备 pairing.
- **Safer execution controls.** `system.run` is gated by 节点 allowlists/approvals on that laptop.
- **More device tools.** Nodes expose `canvas`, `camera`, and `screen` in addition to `system.run`.
- **Local browser automation.** Keep the Gateway on a VPS, but run Chrome locally and relay control
  with the Chrome extension + a 节点 主机 on the laptop.

SSH is fine for ad‑hoc shell access, but 节点 are simpler for ongoing 代理 workflows and
设备 automation.

Docs: [节点](/节点), [节点 CLI](/cli/节点), [Chrome extension](/工具/chrome-extension).

### Should I 安装 on a second laptop or just add a 节点

If you only need **local 工具** (screen/camera/exec) on the second laptop, add it as a
**节点**. That keeps a single Gateway and avoids duplicated 配置. Local 节点 工具 are
currently macOS-only, but we plan to extend them to other OSes.

安装 a second Gateway only when you need **hard isolation** or two fully separate bots.

Docs: [节点](/节点), [节点 CLI](/cli/节点), [Multiple gateways](/Gateway/multiple-gateways).

### Do 节点 run a Gateway 服务

No. Only **one Gateway** should run per 主机 unless you intentionally run isolated 配置文件 (参见 [Multiple gateways](/Gateway/multiple-gateways)). 节点 are peripherals that connect
to the Gateway (iOS/Android 节点, or macOS “节点 mode” in the menubar 应用). For headless 节点
主机 and CLI control, 参见 [节点 主机 CLI](/cli/节点).

A full restart is required for `gateway`, `discovery`, and `canvasHost` changes.

### Is there an API RPC way to apply 配置

Yes. `config.apply` validates + writes the full 配置 and restarts the Gateway as part of the 操作.

### configapply wiped my 配置 How do I recover and avoid this

`config.apply` replaces the **entire 配置**. If you send a partial 对象, everything
else is removed.

Recover:
- Restore from backup (git or a copied `~/.clawdbot/moltbot.json`).
- If you have no backup, re-run `moltbot doctor` and reconfigure 渠道/模型.
- If this was unexpected, 文件 a bug and include your last known 配置 or any backup.
- A local coding 代理 can often reconstruct a working 配置 from 日志 or history.

Avoid it:
- Use `moltbot config set` for small changes.
- Use `moltbot configure` for interactive edits.

Docs: [配置](/cli/配置), [Configure](/cli/configure), [Doctor](/Gateway/doctor).

### Whats a minimal sane 配置 for a first 安装

```json5
{
  agents: { defaults: { workspace: "~/clawd" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

This sets your 工作空间 and restricts who can 触发器 the bot.

### How do I set up Tailscale on a VPS and connect from my Mac

Minimal steps:

1) **安装 + login on the VPS**
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2) **安装 + login on your Mac**
   - Use the Tailscale 应用 and sign in to the same tailnet.
3) **Enable MagicDNS (recommended)**
   - In the Tailscale admin console, enable MagicDNS so the VPS has a stable name.
4) **Use the tailnet hostname**
   - SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

If you want the Control UI without SSH, use Tailscale Serve on the VPS:
```bash
moltbot gateway --tailscale serve
```
This keeps the Gateway bound to loopback and exposes HTTPS via Tailscale. 参见 [Tailscale](/Gateway/tailscale).

### How do I connect a Mac 节点 to a remote Gateway Tailscale Serve

Serve exposes the **Gateway Control UI + WS**. 节点 connect over the same Gateway WS 端点.

Recommended 设置:
1) **Make sure the VPS + Mac are on the same tailnet**.
2) **Use the macOS 应用 in Remote mode** (SSH target can be the tailnet hostname).
   The 应用 will tunnel the Gateway 端口 and connect as a 节点.
3) **Approve the 节点** on the Gateway:
   ```bash
   moltbot nodes pending
   moltbot nodes approve <requestId>
   ```

Docs: [Gateway 协议](/Gateway/协议), [Discovery](/Gateway/discovery), [macOS remote mode](/platforms/mac/remote).

## Env vars and .env loading

### How does Moltbot load 环境 变量

Moltbot reads env vars from the parent 进程 (shell, launchd/systemd, CI, etc.) and additionally loads:

- `.env` from the current working 目录
- a global fallback `.env` from `~/.clawdbot/.env` (aka `$CLAWDBOT_STATE_DIR/.env`)

Neither `.env` 文件 overrides existing env vars.

You can also define inline env vars in 配置 (applied only if missing from the 进程 env):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." }
  }
}
```

参见 [/环境](/环境) for full precedence and sources.

### I started the Gateway via the 服务 and my env vars disappeared What now

Two common fixes:

1) Put the missing keys in `~/.clawdbot/.env` so they’re picked up even when the 服务 doesn’t inherit your shell env.
2) Enable shell import (opt‑in convenience):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

This runs your login shell and imports only missing expected keys (never overrides). Env var equivalents:
`CLAWDBOT_LOAD_SHELL_ENV=1`, `CLAWDBOT_SHELL_ENV_TIMEOUT_MS=15000`.

### I set COPILOTGITHUBTOKEN but 模型 状态 shows Shell env off Why

`moltbot models status` reports whether **shell env import** is 已启用. “Shell env: off”
does **not** mean your env vars are missing - it just means Moltbot won’t load
your login shell automatically.

If the Gateway runs as a 服务 (launchd/systemd), it won’t inherit your shell
环境. Fix by doing one of these:

1) Put the token in `~/.clawdbot/.env`:
   ```
   COPILOT_GITHUB_TOKEN=...
   ```
2) Or enable shell import (`env.shellEnv.enabled: true`).
3) Or add it to your config `env` block (applies only if missing).

Then restart the Gateway and recheck:
```bash
moltbot models status
```

Copilot tokens are read from `COPILOT_GITHUB_TOKEN` (also `GH_TOKEN` / `GITHUB_TOKEN`).
参见 [/concepts/模型-提供商](/concepts/模型-提供商) and [/环境](/环境).

## 会话 & multiple chats

### How do I start a fresh conversation

Send `/new` or `/reset` as a standalone 消息. 参见 [会话 management](/concepts/会话).

### Do 会话 reset automatically if I never send new

Yes. Sessions expire after `session.idleMinutes` (默认 **60**). The **next**
消息 starts a fresh 会话 id for that chat 键. This does not delete
transcripts - it just starts a new 会话.

```json5
{
  session: {
    idleMinutes: 240
  }
}
```

### Is there a way to make a team of Moltbots one CEO and many 代理

Yes, via **multi-代理 routing** and **sub-代理**. You can create one coordinator
代理 and several worker 代理 with their own workspaces and 模型.

That said, this is best seen as a **fun experiment**. It is 令牌 heavy and often
less efficient than using one bot with separate 会话. The typical 模型 we
envision is one bot you talk to, with different 会话 for parallel work. That
bot can also spawn sub-代理 when needed.

Docs: [Multi-代理 routing](/concepts/multi-代理), [Sub-代理](/工具/subagents), [代理 CLI](/cli/代理).

### Why did 上下文 get truncated midtask How do I prevent it

会话 上下文 is limited by the 模型 window. Long chats, large 工具 outputs, or many
文件 can 触发器 compaction or truncation.

What helps:
- Ask the bot to summarize the current 状态 and write it to a 文件.
- Use `/compact` before long tasks, and `/new` when switching topics.
- Keep 重要 上下文 in the 工作空间 and ask the bot to read it back.
- Use sub-代理 for long or parallel work so the main chat stays smaller.
- Pick a 模型 with a larger 上下文 window if this happens often.

### How do I completely reset Moltbot but keep it installed

Use the reset 命令:

```bash
moltbot reset
```

Non-interactive full reset:

```bash
moltbot reset --scope full --yes --non-interactive
```

Then re-run onboarding:

```bash
moltbot onboard --install-daemon
```

Notes:
- The onboarding wizard also offers **Reset** if it sees an existing 配置. 参见 [Wizard](/start/wizard).
- If you used profiles (`--profile` / `CLAWDBOT_PROFILE`), reset each state dir (defaults are `~/.clawdbot-<profile>`).
- Dev reset: `moltbot gateway --dev --reset` (dev-only; wipes dev 配置 + 凭据 + 会话 + 工作空间).

### Im getting 上下文 too large 错误 how do I reset or compact

Use one of these:

- **Compact** (keeps the conversation but summarizes older turns):
  ```
  /compact
  ```
  or `/compact <instructions>` to 指南 the 摘要.

- **Reset** (fresh 会话 ID for the same chat 键):
  ```
  /new
  /reset
  ```

If it keeps happening:
- Enable or tune **session pruning** (`agents.defaults.contextPruning`) to trim old 工具 输出.
- Use a 模型 with a larger 上下文 window.

Docs: [Compaction](/concepts/compaction), [会话 pruning](/concepts/会话-pruning), [会话 management](/concepts/会话).

### Why am I seeing LLM 请求 rejected messagesNcontentXtooluseinput 字段 必需

This is a provider validation error: the model emitted a `tool_use` block without the 必需
`input`. It usually means the 会话 history is stale or corrupted (often after long threads
or a 工具/模式 change).

Fix: start a fresh session with `/new` (standalone 消息).

### Why am I getting heartbeat 消息 every 30 minutes

Heartbeats run every **30m** 默认情况下. Tune or disable them:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h"   // or "0m" to disable
      }
    }
  }
}
```

If `HEARTBEAT.md` exists but is effectively empty (only blank lines and markdown
headers like `# Heading`), Moltbot skips the heartbeat run to save API calls.
If the 文件 is missing, the heartbeat still runs and the 模型 decides what to do.

Per-agent overrides use `agents.list[].heartbeat`. Docs: [Heartbeat](/Gateway/heartbeat).

### Do I need to add a bot account to a WhatsApp group

No. Moltbot runs on **your own account**, so if you’re in the group, Moltbot can 参见 it.
By default, group replies are blocked until you allow senders (`groupPolicy: "allowlist"`).

If you want only **you** to be able to 触发器 group replies:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    }
  }
}
```

### How do I get the JID of a WhatsApp group

选项 1 (fastest): tail 日志 and send a 测试 消息 in the group:

```bash
moltbot logs --follow --json
```

Look for `chatId` (or `from`) ending in `@g.us`, like:
`1234567890-1234567890@g.us`.

选项 2 (if already configured/allowlisted): list groups from 配置:

```bash
moltbot directory groups list --channel whatsapp
```

Docs: [WhatsApp](/渠道/whatsapp), [目录](/cli/目录), [日志](/cli/日志).

### Why doesnt Moltbot reply in a group

Two common causes:
- Mention gating is on (default). You must @mention the bot (or match `mentionPatterns`).
- You configured `channels.whatsapp.groups` without `"*"` and the group isn’t allowlisted.

参见 [Groups](/concepts/groups) and [Group 消息](/concepts/group-消息).

### Do groupsthreads share 上下文 with DMs

Direct chats collapse to the main 会话 默认情况下. Groups/渠道 have their own 会话 keys, and Telegram topics / Discord threads are separate 会话. 参见 [Groups](/concepts/groups) and [Group 消息](/concepts/group-消息).

### How many workspaces and 代理 can I create

No hard limits. Dozens (even hundreds) are fine, but watch for:

- **Disk growth:** sessions + transcripts live under `~/.clawdbot/agents/<agentId>/sessions/`.
- **令牌 cost:** more 代理 means more concurrent 模型 用法.
- **Ops overhead:** per-代理 认证 配置文件, workspaces, and 渠道 routing.

Tips:
- Keep one **active** workspace per agent (`agents.defaults.workspace`).
- Prune old 会话 (delete JSONL or store entries) if disk grows.
- Use `moltbot doctor` to spot stray workspaces and 配置文件 mismatches.

### Can I run multiple bots or chats at the same time Slack and how should I set that up

Yes. Use **Multi‑代理 Routing** to run multiple isolated 代理 and route inbound 消息 by
渠道/account/peer. Slack is supported as a 渠道 and can be bound to specific 代理.

Browser access is powerful but not “do anything a human can” - anti‑bot, CAPTCHAs, and MFA can
still block automation. For the most reliable browser control, use the Chrome extension relay
on the machine that runs the browser (and keep the Gateway anywhere).

Best‑practice 设置:
- Always‑on Gateway 主机 (VPS/Mac mini).
- One 代理 per 角色 (绑定).
- Slack 渠道(s) bound to those 代理.
- Local browser via extension relay (or a 节点) when needed.

Docs: [Multi‑代理 Routing](/concepts/multi-代理), [Slack](/渠道/slack),
[Browser](/工具/browser), [Chrome extension](/工具/chrome-extension), [节点](/节点).

## 模型: defaults, selection, aliases, switching

### What is the 默认 模型

Moltbot’s 默认 模型 is whatever you set as:

```
agents.defaults.model.primary
```

Models are referenced as `provider/model` (example: `anthropic/claude-opus-4-5`). If you omit the provider, Moltbot currently assumes `anthropic` as a temporary deprecation fallback - but you should still **explicitly** set `provider/model`.

### What 模型 do you recommend

**Recommended default:** `anthropic/claude-opus-4-5`.  
**Good alternative:** `anthropic/claude-sonnet-4-5`.  
**Reliable (less character):** `openai/gpt-5.2` - nearly as good as Opus, just less personality.  
**Budget:** `zai/glm-4.7`.

MiniMax M2.1 has its own docs: [MiniMax](/提供商/minimax) and
[Local 模型](/Gateway/local-模型).

规则 of thumb: use the **best 模型 you can afford** for high-stakes work, and a cheaper
模型 for routine chat or summaries. You can route 模型 per 代理 and use sub-代理 to
parallelize long tasks (each sub-代理 consumes 令牌). 参见 [模型](/concepts/模型) and
[Sub-代理](/工具/subagents).

Strong 警告: weaker/over-quantized 模型 are more vulnerable to prompt
injection and unsafe 行为. 参见 [安全](/Gateway/安全).

More 上下文: [模型](/concepts/模型).

### Can I use selfhosted 模型 llamacpp vLLM Ollama

Yes. If your local 服务器 exposes an OpenAI-compatible API, you can point a
custom 提供商 at it. Ollama is supported directly and is the easiest 路径.

安全 注意: smaller or heavily quantized 模型 are more vulnerable to prompt
injection. We strongly recommend **large 模型** for any bot that can use 工具.
If you still want small 模型, enable sandboxing and strict 工具 allowlists.

Docs: [Ollama](/提供商/ollama), [Local 模型](/Gateway/local-模型),
[模型 提供商](/concepts/模型-提供商), [安全](/Gateway/安全),
[Sandboxing](/Gateway/sandboxing).

### How do I switch 模型 without wiping my 配置

Use **模型 命令** or edit only the **模型** fields. Avoid full 配置 replaces.

Safe 选项:
- `/model` in chat (quick, per-会话)
- `moltbot models set ...` (更新 just 模型 配置)
- `moltbot configure --section models` (interactive)
- edit `agents.defaults.model` in `~/.clawdbot/moltbot.json`

Avoid `config.apply` with a partial 对象 unless you intend to replace the whole 配置.
If you did overwrite config, restore from backup or re-run `moltbot doctor` to repair.

Docs: [模型](/concepts/模型), [Configure](/cli/configure), [配置](/cli/配置), [Doctor](/Gateway/doctor).

### What do Clawd Flawd and Krill use for 模型

- **Clawd + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-5`) - 参见 [Anthropic](/提供商/anthropic).
- **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - 参见 [MiniMax](/提供商/minimax).

### How do I switch 模型 on the fly without restarting

Use the `/model` 命令 as a standalone 消息:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

You can list available models with `/model`, `/model list`, or `/model status`.

`/model` (and `/model list`) shows a compact, numbered picker. Select by 数字:

```
/model 3
```

You can also force a specific 认证 配置文件 for the 提供商 (per 会话):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Tip: `/model status` shows which agent is active, which `auth-profiles.json` 文件 is being used, and which 认证 配置文件 will be tried next.
It also shows the configured provider endpoint (`baseUrl`) and API mode (`api`) when available.

**How do I unpin a 配置文件 I set with 配置文件**

Re-run `/model` **without** the `@profile` suffix:

```
/model anthropic/claude-opus-4-5
```

If you want to return to the default, pick it from `/model` (or send `/model <default provider/model>`).
Use `/model status` to confirm which 认证 配置文件 is active.

### Can I use GPT 5.2 for daily tasks and Codex 5.2 for coding

Yes. Set one as 默认 and switch as needed:

- **Quick switch (per session):** `/model gpt-5.2` for daily tasks, `/model gpt-5.2-codex` for coding.
- **Default + switch:** set `agents.defaults.model.primary` to `openai-codex/gpt-5.2`, then switch to `openai-codex/gpt-5.2-codex` when coding (or the other way around).
- **Sub-代理:** route coding tasks to sub-代理 with a different 默认 模型.

参见 [模型](/concepts/模型) and [Slash 命令](/工具/slash-命令).

### Why do I 参见 模型 is not allowed and then no reply

If `agents.defaults.models` is set, it becomes the **allowlist** for `/model` and any
会话 overrides. Choosing a 模型 that isn’t in that list returns:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

That 错误 is returned **instead of** a normal reply. Fix: add the 模型 to
`agents.defaults.models`, remove the allowlist, or pick a model from `/model list`.

### Why do I 参见 Unknown 模型 minimaxMiniMaxM21

This means the **提供商 isn’t configured** (no MiniMax 提供商 配置 or 认证
配置文件 was found), so the 模型 can’t be resolved. A fix for this detection is
in **2026.1.12** (unreleased at the time of writing).

Fix checklist:
1) Upgrade to **2026.1.12** (or run from source `main`), then restart the Gateway.
2) Make sure MiniMax is configured (wizard or JSON), or that a MiniMax API 键
   exists in env/认证 配置文件 so the 提供商 can be injected.
3) Use the exact model id (case‑sensitive): `minimax/MiniMax-M2.1` or
   `minimax/MiniMax-M2.1-lightning`.
4) Run:
   ```bash
   moltbot models list
   ```
   and pick from the list (or `/model list` in chat).

参见 [MiniMax](/提供商/minimax) and [模型](/concepts/模型).

### Can I use MiniMax as my 默认 and OpenAI for complex tasks

Yes. Use **MiniMax as the 默认** and switch 模型 **per 会话** when needed.
Fallbacks are for **errors**, not “hard tasks,” so use `/model` or a separate 代理.

**选项 A: switch per 会话**
```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" }
      }
    }
  }
}
```

Then:
```
/model gpt
```

**选项 B: separate 代理**
- 代理 A 默认: MiniMax
- 代理 B 默认: OpenAI
- Route by agent or use `/agent` to switch

Docs: [模型](/concepts/模型), [Multi-代理 Routing](/concepts/multi-代理), [MiniMax](/提供商/minimax), [OpenAI](/提供商/openai).

### Are opus sonnet gpt builtin shortcuts

Yes. Moltbot ships a few default shorthands (only applied when the model exists in `agents.defaults.models`):

- `opus` → `anthropic/claude-opus-4-5`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

If you set your own alias with the same name, your 值 wins.

### How do I defineoverride 模型 shortcuts aliases

Aliases come from `agents.defaults.models.<modelId>.alias`. 示例:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-5" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" }
      }
    }
  }
}
```

Then `/model sonnet` (or `/<alias>` when supported) resolves to that 模型 ID.

### How do I add 模型 from other 提供商 like OpenRouter or ZAI

OpenRouter (pay‑per‑令牌; many 模型):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} }
    }
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." }
}
```

Z.AI (GLM 模型):

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  },
  env: { ZAI_API_KEY: "..." }
}
```

If you reference a provider/model but the required provider key is missing, you’ll get a runtime auth error (e.g. `No API key found for provider "zai"`).

**No API 键 found for 提供商 after adding a new 代理**

This usually means the **new 代理** has an empty 认证 store. 认证 is per-代理 and
stored in:

```
~/.clawdbot/agents/<agentId>/agent/auth-profiles.json
```

Fix 选项:
- Run `moltbot agents add <id>` and configure 认证 during the wizard.
- Or copy `auth-profiles.json` from the main agent’s `agentDir` into the new agent’s `agentDir`.

Do **not** reuse `agentDir` across 代理; it causes 认证/会话 collisions.

## 模型 failover and “All 模型 failed”

### How does failover work

Failover happens in two stages:

1) **认证 配置文件 rotation** within the same 提供商.
2) **Model fallback** to the next model in `agents.defaults.model.fallbacks`.

Cooldowns apply to failing 配置文件 (exponential backoff), so Moltbot can keep responding even when a 提供商 is rate‑limited or temporarily failing.

### What does this 错误 mean

```
No credentials found for profile "anthropic:default"
```

It means the system attempted to use the auth profile ID `anthropic:default`, but could not find 凭据 for it in the expected 认证 store.

### Fix checklist for No 凭据 found for 配置文件 anthropicdefault

- **Confirm where 认证 配置文件 live** (new vs legacy 路径)
  - Current: `~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`
  - Legacy: `~/.clawdbot/agent/*` (migrated by `moltbot doctor`)
- **Confirm your env var is loaded by the Gateway**
  - If you set `ANTHROPIC_API_KEY` in your shell but run the Gateway via systemd/launchd, it may not inherit it. Put it in `~/.clawdbot/.env` or enable `env.shellEnv`.
- **Make sure you’re editing the correct 代理**
  - Multi‑agent setups mean there can be multiple `auth-profiles.json` 文件.
- **Sanity‑check 模型/认证 状态**
  - Use `moltbot models status` to 参见 configured 模型 and whether 提供商 are authenticated.

**Fix checklist for No 凭据 found for 配置文件 anthropic**

This means the run is pinned to an Anthropic 认证 配置文件, but the Gateway
can’t find it in its 认证 store.

- **Use a 设置-令牌**
  - Run `claude setup-token`, then paste it with `moltbot models auth setup-token --provider anthropic`.
  - If the token was created on another machine, use `moltbot models auth paste-token --provider anthropic`.
- **If you want to use an API 键 instead**
  - Put `ANTHROPIC_API_KEY` in `~/.clawdbot/.env` on the **Gateway 主机**.
  - Clear any pinned order that forces a missing 配置文件:
    ```bash
    moltbot models auth order clear --provider anthropic
    ```
- **Confirm you’re running 命令 on the Gateway 主机**
  - In remote mode, 认证 配置文件 live on the Gateway machine, not your laptop.

### Why did it also 尝试 Google Gemini and fail

If your model config includes Google Gemini as a fallback (or you switched to a Gemini shorthand), Moltbot will try it during model fallback. If you haven’t configured Google credentials, you’ll see `No API key found for provider "google"`.

Fix: either provide Google auth, or remove/avoid Google models in `agents.defaults.model.fallbacks` / aliases so fallback doesn’t route there.

**LLM 请求 rejected 消息 thinking signature 必需 google antigravity**

Cause: the 会话 history contains **thinking blocks without signatures** (often from
an aborted/partial 流). Google Antigravity requires signatures for thinking blocks.

Fix: Moltbot now strips unsigned thinking blocks for Google Antigravity Claude. If it still appears, start a **new session** or set `/thinking off` for that 代理.

## 认证 配置文件: what they are and 如何 manage them

相关: [/concepts/oauth](/concepts/oauth) (OAuth flows, 令牌 存储, multi-account patterns)

### What is an 认证 配置文件

An 认证 配置文件 is a named 凭据 记录 (OAuth or API 键) tied to a 提供商. 配置文件 live in:

```
~/.clawdbot/agents/<agentId>/agent/auth-profiles.json
```

### What are typical 配置文件 IDs

Moltbot uses 提供商‑prefixed IDs like:

- `anthropic:default` (common when no email identity exists)
- `anthropic:<email>` for OAuth identities
- custom IDs you choose (e.g. `anthropic:work`)

### Can I control which 认证 配置文件 is tried first

Yes. Config supports optional metadata for profiles and an ordering per provider (`auth.order.<provider>`). This does **not** store secrets; it maps IDs to 提供商/mode and sets rotation order.

Moltbot may temporarily skip a profile if it’s in a short **cooldown** (rate limits/timeouts/auth failures) or a longer **disabled** state (billing/insufficient credits). To inspect this, run `moltbot models status --json` and check `auth.unusableProfiles`. Tuning: `auth.cooldowns.billingBackoffHours*`.

You can also set a **per-agent** order override (stored in that agent’s `auth-profiles.json`) via the CLI:

```bash
# Defaults to the configured default agent (omit --agent)
moltbot models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
moltbot models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
moltbot models auth order set --provider anthropic anthropic:work anthropic:default

# Clear override (fall back to config auth.order / round-robin)
moltbot models auth order clear --provider anthropic
```

To target a specific 代理:

```bash
moltbot models auth order set --provider anthropic --agent main anthropic:default
```

### OAuth vs API 键 whats the difference

Moltbot supports both:

- **OAuth** often leverages subscription access (where applicable).
- **API keys** use pay‑per‑令牌 billing.

The wizard explicitly supports Anthropic 设置-令牌 and OpenAI Codex OAuth and can store API keys for you.

## Gateway: 端口, “already running”, and remote mode

### What 端口 does the Gateway use

`gateway.port` controls the single multiplexed 端口 for WebSocket + HTTP (Control UI, 钩子, etc.).

Precedence:

```
--port > CLAWDBOT_GATEWAY_PORT > gateway.port > default 18789
```

### Why does moltbot Gateway 状态 say Runtime running but RPC probe failed

Because “running” is the **supervisor’s** view (launchd/systemd/schtasks). The RPC probe is the CLI actually connecting to the gateway WebSocket and calling `status`.

Use `moltbot gateway status` and trust these lines:
- `Probe target:` (the URL the probe actually used)
- `Listening:` (what’s actually bound on the 端口)
- `Last gateway error:` (common root cause when the 进程 is alive but the 端口 isn’t listening)

### Why does moltbot Gateway 状态 show 配置 cli and 配置 服务 different

You’re editing one config file while the service is running another (often a `--profile` / `CLAWDBOT_STATE_DIR` mismatch).

Fix:
```bash
moltbot gateway install --force
```
Run that from the same `--profile` / 环境 you want the 服务 to use.

### What does another Gateway instance is already listening mean

Moltbot enforces a runtime lock by binding the WebSocket listener immediately on startup (default `ws://127.0.0.1:18789`). If the bind fails with `EADDRINUSE`, it throws `GatewayLockError` indicating another instance is already listening.

Fix: stop the other instance, free the port, or run with `moltbot gateway --port <port>`.

### How do I run Moltbot in remote mode 客户端 connects to a Gateway elsewhere

Set `gateway.mode: "remote"` and point to a remote WebSocket URL, optionally with a 令牌/password:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

Notes:
- `moltbot gateway` only starts when `gateway.mode` is `local` (or you pass the override flag).
- The macOS 应用 watches the 配置 文件 and switches modes live when these values change.

### The Control UI says unauthorized or keeps reconnecting What now

Your gateway is running with auth enabled (`gateway.auth.*`), but the UI is not sending the matching 令牌/password.

Facts (from code):
- The Control UI stores the token in browser localStorage key `moltbot.control.settings.v1`.
- The UI can import `?token=...` (and/or `?password=...`) once, then strips it from the URL.

Fix:
- Fastest: `moltbot dashboard` (prints + copies tokenized link, tries to open; shows SSH hint if headless).
- If you don’t have a token yet: `moltbot doctor --generate-gateway-token`.
- If remote, tunnel first: `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/?token=...`.
- Set `gateway.auth.token` (or `CLAWDBOT_GATEWAY_TOKEN`) on the Gateway 主机.
- In the Control UI settings, paste the same token (or refresh with a one-time `?token=...` link).
- Still stuck? Run `moltbot status --all` and follow [故障排除](/Gateway/故障排除). 参见 [Dashboard](/Web/dashboard) for 认证 details.

### I set gatewaybind tailnet but it cant bind nothing listens

`tailnet` bind picks a Tailscale IP from your 网络 interfaces (100.64.0.0/10). If the machine isn’t on Tailscale (or the 接口 is down), there’s nothing to bind to.

Fix:
- Start Tailscale on that 主机 (so it has a 100.x address), or
- Switch to `gateway.bind: "loopback"` / `"lan"`.
  
Note: `tailnet` is explicit. `auto` prefers loopback; use `gateway.bind: "tailnet"` when you want a tailnet-only bind.

### Can I run multiple Gateways on the same 主机

Usually no - one Gateway can run multiple messaging 渠道 and 代理. Use multiple Gateways only when you need redundancy (ex: rescue bot) or hard isolation.

Yes, but you must isolate:

- `CLAWDBOT_CONFIG_PATH` (per‑instance 配置)
- `CLAWDBOT_STATE_DIR` (per‑instance 状态)
- `agents.defaults.workspace` (工作空间 isolation)
- `gateway.port` (unique 端口)

Quick 设置 (recommended):
- Use `moltbot --profile <name> …` per instance (auto-creates `~/.clawdbot-<name>`).
- Set a unique `gateway.port` in each profile config (or pass `--port` for manual runs).
- Install a per-profile service: `moltbot --profile <name> gateway install`.

Profiles also suffix service names (`bot.molt.<profile>`; legacy `com.clawdbot.*`, `moltbot-gateway-<profile>.service`, `Moltbot Gateway (<profile>)`).
Full 指南: [Multiple gateways](/Gateway/multiple-gateways).

### What does invalid handshake code 1008 mean

The Gateway is a **WebSocket 服务器**, and it expects the very first 消息 to
be a `connect` frame. If it receives anything else, it closes the 连接
with **code 1008** (策略 violation).

Common causes:
- You opened the **HTTP** URL in a browser (`http://...`) instead of a WS 客户端.
- You used the wrong 端口 or 路径.
- A proxy or tunnel stripped 认证 headers or sent a non‑Gateway 请求.

Quick fixes:
1) Use the WS URL: `ws://<host>:18789` (or `wss://...` if HTTPS).
2) Don’t open the WS 端口 in a normal browser tab.
3) If auth is on, include the token/password in the `connect` frame.

If you’re using the CLI or TUI, the URL should look like:
```
moltbot tui --url ws://<host>:18789 --token <token>
```

协议 details: [Gateway 协议](/Gateway/协议).

## 日志记录 and 调试

### Where are 日志

文件 日志 (structured):

```
/tmp/moltbot/moltbot-YYYY-MM-DD.log
```

You can set a stable path via `logging.file`. File log level is controlled by `logging.level`. Console verbosity is controlled by `--verbose` and `logging.consoleLevel`.

Fastest 日志 tail:

```bash
moltbot logs --follow
```

服务/supervisor 日志 (when the Gateway runs via launchd/systemd):
- macOS: `$CLAWDBOT_STATE_DIR/logs/gateway.log` and `gateway.err.log` (default: `~/.clawdbot/logs/...`; profiles use `~/.clawdbot-<profile>/logs/...`)
- Linux: `journalctl --user -u moltbot-gateway[-<profile>].service -n 200 --no-pager`
- Windows: `schtasks /Query /TN "Moltbot Gateway (<profile>)" /V /FO LIST`

参见 [故障排除](/Gateway/故障排除#日志-locations) for more.

### How do I startstoprestart the Gateway 服务

Use the Gateway helpers:

```bash
moltbot gateway status
moltbot gateway restart
```

If you run the gateway manually, `moltbot gateway --force` can reclaim the 端口. 参见 [Gateway](/Gateway).

### I closed my terminal on Windows how do I restart Moltbot

There are **two Windows 安装 modes**:

**1) WSL2 (recommended):** the Gateway runs inside Linux.

Open PowerShell, enter WSL, then restart:

```powershell
wsl
moltbot gateway status
moltbot gateway restart
```

If you never installed the 服务, start it in the foreground:

```bash
moltbot gateway run
```

**2) Native Windows (not recommended):** the Gateway runs directly in Windows.

Open PowerShell and run:

```powershell
moltbot gateway status
moltbot gateway restart
```

If you run it manually (no 服务), use:

```powershell
moltbot gateway run
```

Docs: [Windows (WSL2)](/platforms/windows), [Gateway 服务 runbook](/Gateway).

### The Gateway is up but replies never arrive What should I check

Start with a quick health sweep:

```bash
moltbot status
moltbot models status
moltbot channels status
moltbot logs --follow
```

Common causes:
- Model auth not loaded on the **gateway host** (check `models status`).
- 渠道 pairing/allowlist blocking replies (check 渠道 配置 + 日志).
- WebChat/Dashboard is open without the right 令牌.

If you are remote, confirm the tunnel/Tailscale 连接 is up and that the
Gateway WebSocket is reachable.

Docs: [渠道](/渠道), [故障排除](/Gateway/故障排除), [Remote access](/Gateway/remote).

### Disconnected from Gateway no reason what now

This usually means the UI lost the WebSocket 连接. Check:

1) Is the Gateway running? `moltbot gateway status`
2) Is the Gateway healthy? `moltbot status`
3) Does the UI have the right token? `moltbot dashboard`
4) If remote, is the tunnel/Tailscale link up?

Then tail 日志:

```bash
moltbot logs --follow
```

Docs: [Dashboard](/Web/dashboard), [Remote access](/Gateway/remote), [故障排除](/Gateway/故障排除).

### Telegram setMyCommands fails with 网络 错误 What should I check

Start with 日志 and 渠道 状态:

```bash
moltbot channels status
moltbot channels logs --channel telegram
```

If you are on a VPS or behind a proxy, confirm outbound HTTPS is allowed and DNS works.
If the Gateway is remote, make sure you are looking at 日志 on the Gateway 主机.

Docs: [Telegram](/渠道/telegram), [渠道 故障排除](/渠道/故障排除).

### TUI shows no 输出 What should I check

First confirm the Gateway is reachable and the 代理 can run:

```bash
moltbot status
moltbot models status
moltbot logs --follow
```

In the TUI, use `/status` to 参见 the current 状态. If you expect replies in a chat
channel, make sure delivery is enabled (`/deliver on`).

Docs: [TUI](/tui), [Slash 命令](/工具/slash-命令).

### How do I completely stop then start the Gateway

If you installed the 服务:

```bash
moltbot gateway stop
moltbot gateway start
```

This stops/starts the **supervised 服务** (launchd on macOS, systemd on Linux).
Use this when the Gateway runs in the background as a daemon.

If you’re running in the foreground, stop with Ctrl‑C, then:

```bash
moltbot gateway run
```

Docs: [Gateway 服务 runbook](/Gateway).

### ELI5 moltbot Gateway restart vs moltbot Gateway

- `moltbot gateway restart`: restarts the **background 服务** (launchd/systemd).
- `moltbot gateway`: runs the Gateway **in the foreground** for this terminal 会话.

If you installed the service, use the gateway commands. Use `moltbot gateway` when
you want a one-off, foreground run.

### Whats the fastest way to get more details when something fails

Start the Gateway with `--verbose` to get more console detail. Then inspect the 日志 文件 for 渠道 认证, 模型 routing, and RPC 错误.

## Media & attachments

### My 技能 generated an imagePDF but nothing was sent

Outbound attachments from the agent must include a `MEDIA:<path-or-url>` line (on its own line). 参见 [Moltbot assistant 设置](/start/clawd) and [代理 send](/工具/代理-send).

CLI sending:

```bash
moltbot message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

Also check:
- The target 渠道 supports outbound media and isn’t blocked by allowlists.
- The 文件 is within the 提供商’s size limits (images are resized to max 2048px).

参见 [Images](/节点/images).

## 安全 and access control

### Is it safe to expose Moltbot to inbound DMs

Treat inbound DMs as untrusted 输入. Defaults are designed to reduce risk:

- 默认 行为 on DM‑capable 渠道 is **pairing**:
  - Unknown senders receive a pairing code; the bot does not 进程 their 消息.
  - Approve with: `moltbot pairing approve <channel> <code>`
  - Pending requests are capped at **3 per channel**; check `moltbot pairing list <channel>` if a code didn’t arrive.
- Opening DMs publicly requires explicit opt‑in (`dmPolicy: "open"` and allowlist `"*"`).

Run `moltbot doctor` to surface risky DM 策略.

### Is prompt injection only a concern for public bots

No. Prompt injection is about **untrusted content**, not just who can DM the bot.
If your assistant reads external content (Web search/fetch, browser pages, emails,
docs, attachments, pasted 日志), that content can include instructions that 尝试
to hijack the 模型. This can happen even if **you are the only sender**.

The biggest risk is when 工具 are 已启用: the 模型 can be tricked into
exfiltrating 上下文 or calling 工具 on your behalf. Reduce the blast radius by:
- using a read-only or 工具-已禁用 "reader" 代理 to summarize untrusted content
- keeping `web_search` / `web_fetch` / `browser` off for 工具-已启用 代理
- sandboxing and strict 工具 allowlists

Details: [安全](/Gateway/安全).

### Should my bot have its own email GitHub account or phone 数字

Yes, for most setups. Isolating the bot with separate accounts and phone numbers
reduces the blast radius if something goes wrong. This also makes it easier to rotate
凭据 or revoke access without impacting your personal accounts.

Start small. Give access only to the 工具 and accounts you actually need, and expand
later if 必需.

Docs: [安全](/Gateway/安全), [Pairing](/start/pairing).

### Can I give it autonomy over my text 消息 and is that safe

We do **not** recommend full autonomy over your personal 消息. The safest pattern is:
- Keep DMs in **pairing mode** or a tight allowlist.
- Use a **separate 数字 or account** if you want it to 消息 on your behalf.
- Let it draft, then **approve before sending**.

If you want to experiment, do it on a dedicated account and keep it isolated. 参见
[安全](/Gateway/安全).

### Can I use cheaper 模型 for personal assistant tasks

Yes, **if** the 代理 is chat-only and the 输入 is trusted. Smaller tiers are
more susceptible to instruction hijacking, so avoid them for 工具-已启用 代理
or when reading untrusted content. If you must use a smaller 模型, lock down
工具 and run inside a 沙箱. 参见 [安全](/Gateway/安全).

### I ran start in Telegram but didnt get a pairing code

Pairing codes are sent **only** when an unknown sender 消息 the bot and
`dmPolicy: "pairing"` is enabled. `/start` by itself doesn’t generate a code.

Check pending 请求:
```bash
moltbot pairing list telegram
```

If you want immediate access, allowlist your sender id or set `dmPolicy: "open"`
for that account.

### WhatsApp will it 消息 my contacts How does pairing work

No. 默认 WhatsApp DM 策略 is **pairing**. Unknown senders only get a pairing code and their 消息 is **not processed**. Moltbot only replies to chats it receives or to explicit sends you 触发器.

Approve pairing with:

```bash
moltbot pairing approve whatsapp <code>
```

List pending 请求:

```bash
moltbot pairing list whatsapp
```

Wizard phone number prompt: it’s used to set your **allowlist/owner** so your own DMs are permitted. It’s not used for auto-sending. If you run on your personal WhatsApp number, use that number and enable `channels.whatsapp.selfChatMode`.

## Chat 命令, aborting tasks, and “it won’t stop”

### How do I stop internal 系统 消息 from showing in chat

Most internal or 工具 消息 only appear when **verbose** or **reasoning** is 已启用
for that 会话.

Fix in the chat where you 参见 it:
```
/verbose off
/reasoning off
```

If it is still noisy, check the 会话 设置 in the Control UI and set verbose
to **inherit**. Also confirm you are not using a bot profile with `verboseDefault` set
to `on` in 配置.

Docs: [Thinking and verbose](/工具/thinking), [安全](/Gateway/安全#reasoning--verbose-输出-in-groups).

### How do I stopcancel a running task

Send any of these **as a standalone 消息** (no slash):

```
stop
abort
esc
wait
exit
interrupt
```

These are abort 触发 (not slash 命令).

For background 进程 (from the exec 工具), you can ask the 代理 to run:

```
process action:kill sessionId:XXX
```

Slash 命令 概述: 参见 [Slash 命令](/工具/slash-命令).

Most commands must be sent as a **standalone** message that starts with `/`, but a few shortcuts (like `/status`) also work inline for allowlisted senders.

### How do I send a Discord 消息 from Telegram Crosscontext messaging denied

Moltbot blocks **cross‑提供商** messaging 默认情况下. If a 工具 call is bound
to Telegram, it won’t send to Discord unless you explicitly allow it.

Enable cross‑提供商 messaging for the 代理:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " }
          }
        }
      }
    }
  }
}
```

Restart the Gateway after editing 配置. If you only want this for a single
agent, set it under `agents.list[].tools.message` instead.

### Why does it feel like the bot ignores rapidfire 消息

Queue mode controls how new messages interact with an in‑flight run. Use `/queue` to change modes:

- `steer` - new 消息 redirect the current task
- `followup` - run 消息 one at a time
- `collect` - batch 消息 and reply once (默认)
- `steer-backlog` - steer now, then 进程 backlog
- `interrupt` - abort current run and start fresh

You can add options like `debounce:2s cap:25 drop:summarize` for followup modes.

## Answer the exact question from the screenshot/chat 日志

**Q: “What’s the 默认 模型 for Anthropic with an API 键?”**

**A:** In Moltbot, credentials and model selection are separate. Setting `ANTHROPIC_API_KEY` (or storing an Anthropic API key in auth profiles) enables authentication, but the actual default model is whatever you configure in `agents.defaults.model.primary` (for example, `anthropic/claude-sonnet-4-5` or `anthropic/claude-opus-4-5`). If you see `No credentials found for profile "anthropic:default"`, it means the Gateway couldn’t find Anthropic credentials in the expected `auth-profiles.json` for the 代理 that’s running.

---

Still stuck? Ask in [Discord](https://discord.com/invite/clawd) or open a [GitHub discussion](https://github.com/moltbot/moltbot/discussions).
