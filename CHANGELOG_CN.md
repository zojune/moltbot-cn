# 更新日志

文档：https://docs.molt.bot

## 2026.1.27-beta.1
状态：beta。

### 变更
- 品牌重塑：将 npm 包/CLI 重命名为 `moltbot`，添加 `moltbot` 兼容性垫片，并将扩展移动到 `@moltbot/*` 范围。
- 命令：使用 Telegram 分页对 /help 和 /commands 输出进行分组。(#2504) 感谢 @hougangdev。
- macOS：将项目本地 `node_modules/.bin` PATH 首选项限制为调试构建（降低 PATH 劫持风险）。
- macOS：完成 macOS 源代码、捆绑标识符和共享工具包路径的 Moltbot 应用程序重命名。(#2844) 感谢 @fal3。
- 品牌：更新 launchd 标签、移动捆绑 ID 和日志子系统到 bot.molt（遗留 com.clawdbot 迁移）。感谢 @thewilloftheshadow。
- 工具：添加每个发送者的组工具策略并修复优先级。(#1757) 感谢 @adam91holt。
- Agents：在压缩安全保护修剪期间总结丢弃的消息。(#2509) 感谢 @jogi47。
- 技能：为 Nano Banana Pro 技能添加多图像输入支持。(#1958) 感谢 @tyler6204。
- Agents：在 exec 允许列表检查中遵守 tools.exec.safeBins。(#2281)
- Matrix：将插件 SDK 切换到 @vector-im/matrix-bot-sdk。
- 文档：收紧 Fly 私有部署步骤。(#2289) 感谢 @dguido。
- 文档：添加迁移到新机器的指南。(#2381)
- 文档：添加 Northflank 一键部署指南。(#2167) 感谢 @AdeboyeDN。
- Gateway：警告通过查询参数传递的 hook 令牌；记录首选项的 header auth。(#2200) 感谢 @YuriNachos。
- Gateway：添加危险的控制 UI 设备 auth 绕过标志 + 审计警告。(#2248)
- Doctor：警告没有 auth 的网关暴露。(#2016) 感谢 @Alex-Alaniz。
- 配置：自动迁移遗留状态/配置路径，并在遗留文件名之间保持配置解析一致。
- Discord：为存在/成员添加可配置的特权网关意图。(#2266) 感谢 @kentaro。
- 文档：将 Vercel AI Gateway 添加到提供者侧边栏。(#1901) 感谢 @jerilynzheng。
- Agents：使用完整的架构文档扩展 cron 工具描述。(#1988) 感谢 @tomascupr。
- 技能：为 GitHub、Notion、Slack、Discord 添加缺少的依赖元数据。(#1995) 感谢 @jackheuberger。
- 文档：添加 Render 部署指南。(#1975) 感谢 @anurag。
- 文档：添加 Claude Max API 代理指南。(#1875) 感谢 @atalovesyou。
- 文档：添加 DigitalOcean 部署指南。(#1870) 感谢 @0xJonHoldsCrypto。
- 文档：添加 Oracle Cloud (OCI) 平台指南 + 交叉链接。(#2333) 感谢 @hirefrank。
- 文档：添加树莓派安装指南。(#1871) 感谢 @0xJonHoldsCrypto。
- 文档：添加 GCP Compute Engine 部署指南。(#1848) 感谢 @hougangdev。
- 文档：添加 LINE 频道指南。感谢 @thewilloftheshadow。
- 文档：为控制 UI 刷新归功于两位贡献者。(#1852) 感谢 @EnzeD。
- 入门：将 Venice API 密钥添加到非交互式流程。(#1893) 感谢 @jonisjongithub。
- 入门：加强 beta + 访问控制预期的安全警告副本。
- Tlon：将线程回复 ID 格式化为 @ud。(#1837) 感谢 @wca4a。
- Gateway：在组合存储时首选最新的会话元数据。(#1823) 感谢 @emanuelst。
- Web UI：在 WebChat 中保持子代理公告回复可见。(#1977) 感谢 @andrescardonas7。
- CI：为 macOS 检查增加 Node 堆大小。(#1890) 感谢 @realZachi。
- macOS：通过将 Textual 升级到 0.3.1 来避免渲染代码块时崩溃。(#2033) 感谢 @garricn。
- Browser：回退到 URL 匹配以进行扩展中继目标解析。(#1999) 感谢 @jonit-dev。
- Browser：通过网关/节点路由浏览器控制；删除独立的浏览器控制命令和控制 URL 配置。
- Browser：在可用时通过节点代理路由 `browser.request`；遵守代理超时；从 `gateway.port` 派生浏览器端口。
- 更新：忽略 dirty 检查的 dist/control-ui 并在 ui 构建后恢复。(#1976) 感谢 @Glucksberg。
- 构建：在构建期间捆绑 A2UI 资产并停止跟踪生成的捆绑包。(#2455) 感谢 @0oAstro。
- Telegram：允许媒体发送的 caption 参数。(#1888) 感谢 @mguellsegarra。
- Telegram：支持插件 sendPayload channelData（媒体/按钮）并验证插件命令。(#1917) 感谢 @JoshuaLelon。
- Telegram：在禁用流式传输时避免块回复。(#1885) 感谢 @ivancasco。
- 文档：保持文档标题粘性，以便导航栏在滚动时保持可见。(#2445) 感谢 @chenyuan99。
- 文档：更新 exe.dev 安装说明。(#https://github.com/moltbot/moltbot/pull/3047) 感谢 @zackerthescar。
- 安全：使用 Windows ACL 进行 Windows 上的权限审计和修复。(#1957)
- Auth：在 ASCII 提示后显示可复制的 Google auth URL。(#1787) 感谢 @robbyczgw-cla。
- 路由：预编译会话密钥正则表达式。(#1697) 感谢 @Ray0907。
- TUI：在渲染选择列表时避免宽度溢出。(#1686) 感谢 @mossein。
- Telegram：在重启哨兵通知中保留主题 ID。(#1807) 感谢 @hsrvc。
- Telegram：添加可选的静默发送标志（禁用通知）。(#2382) 感谢 @Suksham-sharma。
- Telegram：通过 message(action="edit") 支持编辑发送的消息。(#2394) 感谢 @marcelomar21。
- Telegram：支持消息工具和入站上下文的引用回复。(#2900) 感谢 @aduk059。
- Telegram：添加带有视觉缓存的贴纸接收/发送。(#2629) 感谢 @longjos。
- Telegram：将贴纸像素发送到视觉模型。(#2650)
- 配置：在 ${VAR} 替换之前应用 config.env。(#1813) 感谢 @spanishflu-est1918。
- Slack：在流式回复后清除 ack 反应。(#2044) 感谢 @fancyboi999。
- macOS：在远程目标中保留自定义 SSH 用户名。(#2046) 感谢 @algal。
- CLI：使用 Node 的模块编译缓存以加快启动速度。(#2808) 感谢 @pi0。
- 路由：添加每账户 DM 会话范围并记录多账户隔离。(#3095) 感谢 @jarvis-sam。

### 破坏性变更
- **破坏性变更：** Gateway auth 模式 "none" 已被删除；网关现在需要令牌/密码（仍允许 Tailscale Serve 身份）。

### 修复
- Discord：在目标解析中恢复用户名目录查找。(#3131) 感谢 @bonald。
- Agents：使 MiniMax 基础 URL 测试期望与默认提供者配置保持一致。(#3131) 感谢 @bonald。
- Agents：防止对超大图像错误进行重试并显示大小限制。(#2871) 感谢 @Suksham-sharma。
- Agents：为内联模型继承提供者 baseUrl/api。(#2740) 感谢 @lploc94。
- Memory Search：保持自动提供者模型默认值，仅在配置时包括远程。(#2576) 感谢 @papago2355。
- macOS：在向上滚动时发送新消息时自动滚动到底部。(#2471) 感谢 @kennyklee。
- Web UI：在键入时自动展开聊天撰写文本区域（具有合理的最大高度）。(#2950) 感谢 @shivamraut101。
- Gateway：防止瞬态网络错误（获取失败、超时、DNS）导致的崩溃。添加了致命错误检测，仅在真正的关键错误时退出。修复 #2895、#2879、#2873。(#2980) 感谢 @elliotsecops。
- Agents：保护频道工具列表以避免插件崩溃。(#2859) 感谢 @mbelinky。
- Discord：停止 resolveDiscordTarget 将目录参数传递到消息目标解析器。修复 #3167。感谢 @thewilloftheshadow。
- Discord：避免在用户名匹配时将裸频道名称解析为用户 DM。感谢 @thewilloftheshadow。
- Discord：修复目标解析的目录配置类型导入。感谢 @thewilloftheshadow。
- 提供者：更新 MiniMax API 端点和兼容性模式。(#3064) 感谢 @hlbbbbbbb。
- Telegram：在轮询中将更多网络错误视为可恢复的。(#3013) 感谢 @ryancontent。
- Discord：为出站消息将用户名解析为用户 ID。(#2649) 感谢 @nonggialiang。
- 提供者：将 Moonshot Kimi 模型引用更新为 kimi-k2.5。(#2762) 感谢 @MarvinCui。
- Gateway：在未处理的拒绝中抑制 AbortError 和瞬态网络错误。(#2451) 感谢 @Glucksberg。
- TTS：在仅文本命令上保留 /tts 状态回复并避免重复的块流音频。(#2451) 感谢 @Glucksberg。
- 安全：将 npm 覆盖固定为 tar@7.5.4 以用于安装工具链。
- 安全：正确测试配置包含的 Windows ACL 审计。(#2403) 感谢 @dominicnunez。
- CLI：在解析 argv 时识别版本化的 Node 可执行文件。(#2490) 感谢 @David-Marsh-Photo。
- CLI：避免在微调器下提示网关运行时。(#2874)
- BlueBubbles：合并入站 URL 链接预览消息。(#1981) 感谢 @tyler6204。
- Cron：允许在事件过滤器中包含 "heartbeat" 的有效负载。(#2219) 感谢 @dwfinkelstein。
- CLI：在注册插件命令时避免为全局帮助/版本加载配置。(#2212) 感谢 @dial481。
- Agents：在引导内存上下文时包括 memory.md。(#2318) 感谢 @czekaj。
- Agents：在进程终止时释放会话锁并覆盖更多信号。(#2483) 感谢 @janeexai。
- Agents：在模型故障转移期间跳过冷却的提供者。(#2143) 感谢 @YiWang24。
- Telegram：针对瞬态网络错误和 Node 22 传输问题加强轮询 + 重试行为。(#2420) 感谢 @techboss。
- Telegram：在保留 DM 线程会话的同时，忽略非论坛组的 message_thread_id。(#2731) 感谢 @dylanneve1。
- Telegram：每行包装推理斜体，以避免原始下划线。(#2181) 感谢 @YuriNachos。
- Telegram：集中交付和 bot 调用的 API 错误记录。(#2492) 感谢 @altryne。
- Voice Call：为 ngrok URL 执行 Twilio webhook 签名验证；默认禁用 ngrok 免费层绕过。
- 安全：通过在信任标头之前通过本地 tailscaled 验证身份来加强 Tailscale Serve auth。
- 构建：使 memory-core 对等依赖与锁文件对齐。
- 安全：添加 mDNS 发现模式，具有最小默认值以减少信息泄露。(#1882) 感谢 @orlyjamie。
- 安全：使用 DNS 固定加强 URL 获取以减少重新绑定风险。感谢 Chris Zheng。
- Web UI：改进 WebChat 图像粘贴预览并允许仅图像发送。(#1925) 感谢 @smartprogrammer93。
- 安全：默认使用每个 hook 可选择退出包装外部 hook 内容。(#1827) 感谢 @mertcicekci0。
- Gateway：默认 auth 现在是 fail-closed（需要令牌/密码；Tailscale Serve 身份仍然允许）。
- Gateway：将环回 + 非本地主机连接视为远程，除非存在受信任的代理标头。
- 入门：从入门/配置流程和 CLI 标志中删除不支持的网关 auth "off" 选择。

## 2026.1.24-3

### 修复
- Slack：修复由于跨源重定向上缺少 Authorization 标头而导致的图像下载失败。(#1936) 感谢 @sanderhelgesen。
- Gateway：加强本地客户端检测和未经身份验证的代理连接的反向代理处理。(#1795) 感谢 @orlyjamie。
- 安全审计：将禁用 auth 的环回控制 UI 标记为关键。(#1795) 感谢 @orlyjamie。
- CLI：恢复 claude-cli 会话并将 CLI 回复流式传输到 TUI 客户端。(#1921) 感谢 @rmorse。

## 2026.1.24-2

### 修复
- 打包：在 npm tarball 中包括 dist/link-understanding 输出（修复安装时缺少的 apply.js 导入）。

## 2026.1.24-1

### 修复
- 打包：在 npm tarball 中包括 dist/shared 输出（修复安装时缺少的 reasoning-tags 导入）。

## 2026.1.24

### 亮点
- 提供者：Ollama 发现 + 文档；Venice 指南升级 + 交叉链接。(#1606) 感谢 @abhaymundhara。https://docs.molt.bot/providers/ollama https://docs.molt.bot/providers/venice
- 频道：LINE 插件（消息传递 API），具有丰富的回复 + 快速回复。(#1630) 感谢 @plum-dawg。
- TTS：Edge 回退（无密钥）+ `/tts` 自动模式。(#1668、#1667) 感谢 @steipete、@sebslight。https://docs.molt.bot/tts
- Exec 批准：通过所有频道的 `/approve` 在聊天中批准（包括插件）。(#1621) 感谢 @czekaj。https://docs.molt.bot/tools/exec-approvals https://docs.molt.bot/tools/slash-commands
- Telegram：将 DM 主题作为单独的会话 + 出站链接预览切换。(#1597、#1700) 感谢 @rohannagpal、@zerone0x。https://docs.molt.bot/channels/telegram

### 变更
- 频道：添加 LINE 插件（消息传递 API），具有丰富的回复、快速回复和插件 HTTP 注册表。(#1630) 感谢 @plum-dawg。
- TTS：添加 Edge TTS 提供者回退，默认为无密钥 Edge，在格式失败时使用 MP3 重试。(#1668) 感谢 @steipete。https://docs.molt.bot/tts
- TTS：添加自动模式枚举（off/always/inbound/tagged），具有每会话 `/tts` 覆盖。(#1667) 感谢 @sebslight。https://docs.molt.bot/tts
- Telegram：将 DM 主题视为单独的会话，并使用线程后缀保持 DM 历史限制稳定。(#1597) 感谢 @rohannagpal。
- Telegram：添加 `channels.telegram.linkPreview` 来切换出站链接预览。(#1700) 感谢 @zerone0x。https://docs.molt.bot/channels/telegram
- Web 搜索：为时间范围结果添加 Brave 新鲜度过滤参数。(#1688) 感谢 @JonUleis。https://docs.molt.bot/tools/web
- UI：刷新控制 UI 仪表板设计系统（颜色、图标、排版）。(#1745、#1786) 感谢 @EnzeD、@mousberg。
- Exec 批准：通过所有频道的 `/approve` 将批准提示转发到聊天（包括插件）。(#1621) 感谢 @czekaj。https://docs.molt.bot/tools/exec-approvals https://docs.molt.bot/tools/slash-commands
- Gateway：在网关工具中公开 config.patch，具有安全的部分更新 + 重启哨兵。(#1653) 感谢 @Glucksberg。
- 诊断：为有针对性的调试日志添加诊断标志（配置 + 环境覆盖）。https://docs.molt.bot/diagnostics/flags
- 文档：扩展 FAQ（迁移、调度、并发、模型推荐、OpenAI 订阅 auth、Pi 大小、可黑客安装、docs SSL 变通方法）。
- 文档：添加详细的安装程序故障排除指南。
- 文档：添加 macOS VM 指南，具有本地/托管选项 + VPS/节点指南。(#1693) 感谢 @f-trycua。
- 文档：添加 Bedrock EC2 实例角色设置 + IAM 步骤。(#1625) 感谢 @sergical。https://docs.molt.bot/bedrock
- 文档：更新 Fly.io 指南说明。
- 开发：添加 prek 提交前钩子 + dependabot 配置以进行每周更新。(#1720) 感谢 @dguido。

### 修复
- Web UI：修复配置/调试布局溢出、滚动和代码块大小。(#1715) 感谢 @saipreetham589。
- Web UI：在活动运行期间显示停止按钮，在空闲时切换回新会话。(#1664) 感谢 @ndbroadbent。
- Web UI：在重新连接时清除过时的断开连接横幅；允许表单保存不支持的架构路径，但在缺少架构时阻止。(#1707) 感谢 @Glucksberg。
- Web UI：在聊天气泡中隐藏内部 `message_id` 提示。
- Gateway：即使存在设备身份，也允许控制 UI 仅令牌 auth 跳过设备配对（`gateway.controlUi.allowInsecureAuth`）。(#1679) 感谢 @steipete。
- Matrix：使用预检大小保护解密 E2EE 媒体附件。(#1744) 感谢 @araa47。
- BlueBubbles：将电话号码目标路由到 DM，避免泄露路由 ID，并自动创建缺失的 DM（需要私有 API）。(#1751) 感谢 @tyler6204。https://docs.molt.bot/channels/bluebubbles
- BlueBubbles：在缺少短 ID 时，在回复标签中保留部分索引 GUID。
- iMessage：不区分大小写地规范化 chat_id/chat_guid/chat_identifier 前缀，并保持服务前缀的处理程序稳定。(#1708) 感谢 @aaronn。
- Signal：修复反应发送（组/UUID 目标 + CLI 作者标志）。(#1651) 感谢 @vilkasdev。
- Signal：添加可配置的 signal-cli 启动超时 + 外部守护进程模式文档。(#1677) https://docs.molt.bot/channels/signal
- Telegram：在 Node 22 上为上传设置 fetch duplex="half" 以避免 sendPhoto 失败。(#1684) 感谢 @commdata2338。
- Telegram：在 Node 上为长轮询使用包装的 fetch 以规范化 AbortSignal 处理。(#1639)
- Telegram：为出站 API 调用遵守每账户代理。(#1774) 感谢 @radek-paclt。
- Telegram：当隐私设置阻止语音笔记时回退到文本。(#1725) 感谢 @foeken。
- Voice Call：在初始 Twilio webhook 上为出站对话调用返回流式 TwiML。(#1634)
- Voice Call：序列化 Twilio TTS 播放并在闯入时取消以防止重叠。(#1713) 感谢 @dguido。
- Google Chat：收紧电子邮件允许列表匹配、输入清理、媒体上限和入门/文档/测试。(#1635) 感谢 @iHildy。
- Google Chat：在没有双 `spaces/` 前缀的情况下规范化空间目标。
- Agents：在失败之前自动压缩上下文溢出提示错误。(#1627) 感谢 @rodrigouroz。
- Agents：使用活动的 auth 配置文件进行自动压缩恢复。
- 媒体理解：当主要模型已经支持视觉时跳过图像理解。(#1747) 感谢 @tyler6204。
- 模型：默认缺少的自定义提供者字段，以便接受最小配置。
- 消息：保持换行符分块对跨频道的围栏 markdown 块安全。
- 消息：将换行符分块视为段落感知（空行拆分），以保持列表和标题在一起。(#1726) 感谢 @tyler6204。
- TUI：在网关重新连接后重新加载历史记录以恢复会话状态。(#1663)
- 心跳：规范化目标标识符以进行一致的路由。
- Exec：保留提升 ask 的批准，除非完全模式。(#1616) 感谢 @ivancasco。
- Exec：将 Windows 平台标签视为节点 shell 选择时的 Windows。(#1760) 感谢 @ymat19。
- Gateway：在服务安装环境中包括内联配置环境变量。(#1735) 感谢 @Seredeep。
- Gateway：当 tailscale.mode 关闭时跳过 Tailscale DNS 探测。(#1671)
- Gateway：减少后期调用的日志噪音 + 远程节点探测； debounce 技能刷新。(#1607) 感谢 @petter-b。
- Gateway：阐明缺失令牌的控制 UI/WebChat auth 错误提示。(#1690)
- Gateway：在绑定到 127.0.0.1 时监听 IPv6 环回，以便 localhost webhook 工作。
- Gateway：将锁文件存储在临时目录中，以避免持久卷上的陈旧锁。(#1676)
- macOS：默认直接传输 `ws://` URL 到端口 18789；记录 `gateway.remote.transport`。(#1603) 感谢 @ngutman。
- 测试：在 CI macOS 上限制 Vitest 工作线程以减少超时。(#1597) 感谢 @rohannagpal。
- 测试：在嵌入式运行器流模拟中避免 fake-timer 依赖以减少 CI 片状。(#1597) 感谢 @rohannagpal。
- 测试：增加嵌入式运行器排序测试超时以减少 CI 片状。(#1597) 感谢 @rohannagpal。

[... 由于篇幅限制，以下是更早版本的摘要 ...]

## 更早版本的亮点摘要

### 2026.1.23
- TTS：将 Telegram TTS 移至核心 + 默认启用模型驱动的 TTS 标签以表达音频回复。
- Gateway：添加 `/tools/invoke` HTTP 端点以进行直接工具调用（强制执行 auth + 工具策略）。
- 心跳：每频道可见性控制（OK/警报/指示器）。
- 部署：添加 Fly.io 部署支持 + 指南。
- 频道：添加 Tlon/Urbit 频道插件（DM、组提及、线程回复）。

### 2026.1.22
- 压缩安全保护现在使用自适应分块、渐进式回退和 UI 状态 + 重试。
- 提供者：添加 Antigravity 使用跟踪到状态输出。
- Slack：通过 `replyToModeByChatType` 添加聊天类型回复线程覆盖。
- BlueBubbles：在 sendAttachment 中添加 MP3/CAF 语音备忘录的 `asVoice` 支持。
- 入门：添加孵化选择（TUI/Web/稍后）、令牌说明器、macOS 上的后台仪表板种子和展示链接。

### 2026.1.21
- 重点：Lobster 可选插件工具，用于类型化工作流 + 批准门控。
- 心跳：允许在显式会话密钥中运行心跳。
- CLI：将 exec 批准默认为本地主机，添加网关/节点定位标志，并在允许列表输出中显示目标详细信息。
- 会话：通过 `session.resetByChannel` 添加每频道重置覆盖。
- Agents：添加身份头像配置支持和控制 UI 头像渲染。
- UI：在控制 UI 中显示每会话助手身份。
- CLI：添加用于交互式频道选择的 `moltbot update wizard` 和重新启动提示。
- Signal：添加通过 signal-cli 的输入指示器和 DM 读取回执。
- MSTeams：添加文件上传、自适应卡和附件处理改进。
- 入门：删除运行 setup-token auth 选项（改为粘贴 setup-token 或重用 CLI 凭据）。

### 2026.1.20
- 控制UI：添加复制为markdown，带有错误反馈
- TUI：添加代码块的语法高亮、会话选择器、可搜索模型选择器、输入历史
- ACP：添加IDE集成的`moltbot acp`命令
- Memory：添加混合BM25 + 向量搜索(FTS5)、SQLite嵌入缓存、OpenAI批量索引
- Nostr：添加Nostr频道插件
- Matrix：迁移到支持E2EE的matrix-bot-sdk
- 插件：需要清单中嵌入的配置架构，将频道目录元数据迁移到插件清单
- Agents：添加代理头像支持到身份配置、IDENTITY.md和控制UI

### 安全提示
- 默认：工具在**main**会话的主机上运行，因此当只有你时，代理具有完全访问权限
- 组/频道安全：设置`agents.defaults.sandbox.mode: "non-main"`以在每会话Docker沙箱中运行**非主会话**（组/频道）
- 沙箱默认值：允许列表`bash`、`process`、`read`、`write`、`edit`、`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`；拒绝列表`browser`、`canvas`、`nodes`、`cron`、`discord`、`gateway`

（完整日志请参阅英文版 CHANGELOG.md）
