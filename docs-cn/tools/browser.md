---
summary: "集成浏览器控制服务 + 动作命令"
read_when:
  - 添加代理控制的浏览器自动化
  - 调试 clawd 为何干扰您的 Chrome
  - 在 macOS 应用中实现浏览器设置 + 生命周期
---

# 浏览器（clawd 托管）

Moltbot 可以运行一个**专用的 Chrome/Brave/Edge/Chromium 配置文件**，由代理控制。
它与您的个人浏览器隔离，并通过网关内部的小型本地控制服务进行管理（仅回环）。

初学者视图：
- 将其视为一个**独立的、仅限代理的浏览器**。
- `clawd` 配置文件**不会**触碰您的个人浏览器配置文件。
- 代理可以在安全通道中**打开标签页、阅读页面、点击和输入**。
- 默认 `chrome` 配置文件通过扩展中继使用**系统默认 Chromium 浏览器**；切换到 `clawd` 以获得隔离的托管浏览器。

## 您将获得

- 一个名为 **clawd** 的独立浏览器配置文件（默认为橙色强调）。
- 确定性的标签页控制（列表/打开/聚焦/关闭）。
- 代理动作（点击/输入/拖动/选择）、快照、截图、PDF。
- 可选的多配置文件支持（`clawd`、`work`、`remote`、...）。

此浏览器**不是**您的日常驱动程序。它是用于代理自动化和验证的安全、隔离的界面。

## 快速开始

```bash
moltbot browser --browser-profile clawd status
moltbot browser --browser-profile clawd start
moltbot browser --browser-profile clawd open https://example.com
moltbot browser --browser-profile clawd snapshot
```

如果出现"浏览器已禁用"，请在配置中启用（见下文）并重启网关。

## 配置文件：`clawd` vs `chrome`

- `clawd`：托管、隔离浏览器（无需扩展）。
- `chrome`：到您的**系统浏览器**的扩展中继（需要 Moltbot 扩展附加到标签页）。

如果您希望默认使用托管模式，请设置 `browser.defaultProfile: "clawd"`。

## 配置

浏览器设置位于 `~/.clawdbot/moltbot.json` 中。

```json5
{
  browser: {
    enabled: true,                    // 默认：true
    // cdpUrl: "http://127.0.0.1:18792", // 传统单配置文件覆盖
    remoteCdpTimeoutMs: 1500,         // 远程 CDP HTTP 超时（毫秒）
    remoteCdpHandshakeTimeoutMs: 3000, // 远程 CDP WebSocket 握手超时（毫秒）
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      clawd: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

说明：
- 浏览器控制服务绑定到从 `gateway.port` 派生的端口的回环（默认：`18791`，即网关 + 2）。中继使用下一个端口（`18792`）。
- 如果您覆盖网关端口（`gateway.port` 或 `CLAWDBOT_GATEWAY_PORT`），派生的浏览器端口会移动以保持在相同的"系列"中。
- `cdpUrl` 在未设置时默认为中继端口。
- `remoteCdpTimeoutMs` 适用于远程（非回环）CDP 可达性检查。
- `remoteCdpHandshakeTimeoutMs` 适用于远程 CDP WebSocket 可达性检查。
- `attachOnly: true` 意味着"永不启动本地浏览器；仅在已经运行时附加"。
- `color` + 每配置文件的 `color` 会为浏览器 UI 着色，以便您可以看到哪个配置文件处于活动状态。
- 默认配置文件是 `chrome`（扩展中继）。使用 `defaultProfile: "clawd"` 获得托管浏览器。
- 自动检测顺序：如果基于 Chromium 则为系统默认浏览器；否则为 Chrome → Brave → Edge → Chromium → Chrome Canary。
- 本地 `clawd` 配置文件自动分配 `cdpPort`/`cdpUrl` — 仅为远程 CDP 设置这些选项。

## 使用 Brave（或其他基于 Chromium 的浏览器）

如果您的**系统默认**浏览器是基于 Chromium 的（Chrome/Brave/Edge 等），Moltbot 会自动使用它。设置 `browser.executablePath` 覆盖自动检测：

CLI 示例：

```bash
moltbot config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## 本地 vs 远程控制

- **本地控制（默认）：** 网关启动回环控制服务并可以启动本地浏览器。
- **远程控制（节点主机）：** 在具有浏览器的机器上运行节点主机；网关将浏览器动作代理到它。
- **远程 CDP：** 设置 `browser.profiles.<name>.cdpUrl`（或 `browser.cdpUrl`）以附加到远程基于 Chromium 的浏览器。在这种情况下，Moltbot 不会启动本地浏览器。

远程 CDP URL 可以包含身份验证：
- 查询令牌（例如 `https://provider.example?token=<token>`）
- HTTP Basic 身份验证（例如 `https://user:pass@provider.example`）

Moltbot 在调用 `/json/*` 端点和连接到 CDP WebSocket 时保留身份验证。优先使用环境变量或密钥管理器来存储令牌，而不是将它们提交到配置文件中。

## 节点浏览器代理（零配置默认）

如果您在具有浏览器的机器上运行**节点主机**，Moltbot 可以自动将浏览器工具调用路由到该节点，而无需任何额外的浏览器配置。
这是远程网关的默认路径。

说明：
- 节点主机通过**代理命令**公开其本地浏览器控制服务器。
- 配置文件来自节点自己的 `browser.profiles` 配置（与本地相同）。
- 如果您不想要它，请禁用：
  - 在节点上：`nodeHost.browserProxy.enabled=false`
  - 在网关上：`gateway.nodes.browser.mode="off"`

## Browserless（托管的远程 CDP）

[Browserless](https://browserless.io) 是一个通过 HTTPS 公开 CDP 端点的托管 Chromium 服务。您可以将 Moltbot 浏览器配置文件指向 Browserless 区域端点并使用您的 API 密钥进行身份验证。

示例：
```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

说明：
- 将 `<BROWSERLESS_API_KEY>` 替换为您的真实 Browserless 令牌。
- 选择与您的 Browserless 帐户匹配的区域端点（请参阅其文档）。

## 安全性

关键概念：
- 浏览器控制仅限回环；访问通过网关的身份验证或节点配对流动。
- 将网关和任何节点主机保持在专用网络（Tailscale）上；避免公开暴露。
- 将远程 CDP URL/令牌视为机密；优先使用环境变量或密钥管理器。

远程 CDP 提示：
- 尽可能优先使用 HTTPS 端点和短期令牌。
- 避免将长期令牌直接嵌入配置文件中。

## 配置文件（多浏览器）

Moltbot 支持多个命名配置文件（路由配置）。配置文件可以是：
- **clawd 托管**：具有自己的用户数据目录 + CDP 端口的专用基于 Chromium 的浏览器实例
- **远程**：显式 CDP URL（在其他地方运行的基于 Chromium 的浏览器）
- **扩展中继**：通过本地中继 + Chrome 扩展的现有 Chrome 标签页

默认值：
- 如果缺少，`clawd` 配置文件会自动创建。
- `chrome` 配置文件是为 Chrome 扩展中继内置的（默认指向 `http://127.0.0.1:18792`）。
- 本地 CDP 端口默认从 **18800–18899** 分配。
- 删除配置文件会将其本地数据目录移动到废纸篓。

所有控制端点都接受 `?profile=<name>`；CLI 使用 `--browser-profile`。

## Chrome 扩展中继（使用您现有的 Chrome）

Moltbot 还可以通过本地 CDP 中继 + Chrome 扩展来驱动**您现有的 Chrome 标签页**（无需单独的"clawd" Chrome 实例）。

完整指南：[Chrome 扩展](/tools/chrome-extension)

流程：
- 网关在本地运行（同一机器）或在浏览器机器上运行节点主机。
- 本地**中继服务器**在回环 `cdpUrl` 上侦听（默认：`http://127.0.0.1:18792`）。
- 您点击标签页上的 **Moltbot 浏览器中继**扩展图标进行附加（它不会自动附加）。
- 代理通过选择正确的配置文件，使用正常的 `browser` 工具界面控制该标签页。

如果网关在其他地方运行，请在浏览器机器上运行节点主机，以便网关可以代理浏览器动作。

### 沙箱会话

如果代理会话处于沙箱状态，`browser` 工具可能默认为 `target="sandbox"`（沙箱浏览器）。
Chrome 扩展中继接管需要主机浏览器控制，因此要么：
- 在非沙箱状态下运行会话，或
- 设置 `agents.defaults.sandbox.browser.allowHostControl: true` 并在调用工具时使用 `target="host"`。

### 设置

1) 加载扩展（开发/未打包）：

```bash
moltbot browser extension install
```

- Chrome → `chrome://extensions` → 启用"开发者模式"
- "加载未打包" → 选择 `moltbot browser extension path` 打印的目录
- 固定扩展，然后点击您想要控制的标签页上的它（徽章显示 `ON`）。

2) 使用它：
- CLI：`moltbot browser --browser-profile chrome tabs`
- 代理工具：`browser` 与 `profile="chrome"`

可选：如果您想要不同的名称或中继端口，请创建自己的配置文件：

```bash
moltbot browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

说明：
- 此模式大多数操作（截图/快照/动作）依赖于基于 CDP 的 Playwright。
- 通过再次点击扩展图标来分离。

## 隔离保证

- **专用用户数据目录**：永不触碰您的个人浏览器配置文件。
- **专用端口**：避免 `9222` 以防止与开发工作流冲突。
- **确定性标签页控制**：通过 `targetId` 定向标签页，而不是"最后标签页"。

## 浏览器选择

在本地启动时，Moltbot 选择第一个可用的：
1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

您可以通过 `browser.executablePath` 覆盖。

平台：
- macOS：检查 `/Applications` 和 `~/Applications`。
- Linux：查找 `google-chrome`、`brave`、`microsoft-edge`、`chromium` 等。
- Windows：检查常见安装位置。

## 控制 API（可选）

仅用于本地集成，网关公开一个小型回环 HTTP API：

- 状态/启动/停止：`GET /`、`POST /start`、`POST /stop`
- 标签页：`GET /tabs`、`POST /tabs/open`、`POST /tabs/focus`、`DELETE /tabs/:targetId`
- 快照/截图：`GET /snapshot`、`POST /screenshot`
- 动作：`POST /navigate`、`POST /act`
- 钩子：`POST /hooks/file-chooser`、`POST /hooks/dialog`
- 下载：`POST /download`、`POST /wait/download`
- 调试：`GET /console`、`POST /pdf`
- 调试：`GET /errors`、`GET /requests`、`POST /trace/start`、`POST /trace/stop`、`POST /highlight`
- 网络：`POST /response/body`
- 状态：`GET /cookies`、`POST /cookies/set`、`POST /cookies/clear`
- 状态：`GET /storage/:kind`、`POST /storage/:kind/set`、`POST /storage/:kind/clear`
- 设置：`POST /set/offline`、`POST /set/headers`、`POST /set/credentials`、`POST /set/geolocation`、`POST /set/media`、`POST /set/timezone`、`POST /set/locale`、`POST /set/device`

所有端点都接受 `?profile=<name>`。

### Playwright 要求

某些功能（导航/动作/AI 快照/角色快照、元素截图、PDF）需要 Playwright。如果未安装 Playwright，这些端点会返回清晰的 501 错误。对于 clawd 托管的 Chrome，ARIA 快照和基本截图仍然有效。对于 Chrome 扩展中继驱动程序，ARIA 快照和截图需要 Playwright。

如果您看到"此网关构建中不可用 Playwright"，请安装完整的 Playwright 软件包（而不是 `playwright-core`）并重启网关，或使用浏览器支持重新安装 Moltbot。

## 工作原理（内部）

高级流程：
- 一个小型**控制服务器**接受 HTTP 请求。
- 它通过 **CDP** 连接到基于 Chromium 的浏览器（Chrome/Brave/Edge/Chromium）。
- 对于高级操作（点击/输入/快照/PDF），它在 CDP 之上使用 **Playwright**。
- 当缺少 Playwright 时，仅非 Playwright 操作可用。

这种设计使代理保持在稳定、确定性的界面上，同时允许您交换本地/远程浏览器和配置文件。

## CLI 快速参考

所有命令都接受 `--browser-profile <name>` 以定向到特定配置文件。
所有命令还接受 `--json` 以获得机器可读的输出（稳定载荷）。

基础：
- `moltbot browser status`
- `moltbot browser start`
- `moltbot browser stop`
- `moltbot browser tabs`
- `moltbot browser tab`
- `moltbot browser tab new`
- `moltbot browser tab select 2`
- `moltbot browser tab close 2`
- `moltbot browser open https://example.com`
- `moltbot browser focus abcd1234`
- `moltbot browser close abcd1234`

检查：
- `moltbot browser screenshot`
- `moltbot browser screenshot --full-page`
- `moltbot browser screenshot --ref 12`
- `moltbot browser screenshot --ref e12`
- `moltbot browser snapshot`
- `moltbot browser snapshot --format aria --limit 200`
- `moltbot browser snapshot --interactive --compact --depth 6`
- `moltbot browser snapshot --efficient`
- `moltbot browser snapshot --labels`
- `moltbot browser snapshot --selector "#main" --interactive`
- `moltbot browser snapshot --frame "iframe#main" --interactive`
- `moltbot browser console --level error`
- `moltbot browser errors --clear`
- `moltbot browser requests --filter api --clear`
- `moltbot browser pdf`
- `moltbot browser responsebody "**/api" --max-chars 5000`

动作：
- `moltbot browser navigate https://example.com`
- `moltbot browser resize 1280 720`
- `moltbot browser click 12 --double`
- `moltbot browser click e12 --double`
- `moltbot browser type 23 "hello" --submit`
- `moltbot browser press Enter`
- `moltbot browser hover 44`
- `moltbot browser scrollintoview e12`
- `moltbot browser drag 10 11`
- `moltbot browser select 9 OptionA OptionB`
- `moltbot browser download e12 /tmp/report.pdf`
- `moltbot browser waitfordownload /tmp/report.pdf`
- `moltbot browser upload /tmp/file.pdf`
- `moltbot browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
- `moltbot browser dialog --accept`
- `moltbot browser wait --text "Done"`
- `moltbot browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
- `moltbot browser evaluate --fn '(el) => el.textContent' --ref 7`
- `moltbot browser highlight e12`
- `moltbot browser trace start`
- `moltbot browser trace stop`

状态：
- `moltbot browser cookies`
- `moltbot browser cookies set session abc123 --url "https://example.com"`
- `moltbot browser cookies clear`
- `moltbot browser storage local get`
- `moltbot browser storage local set theme dark`
- `moltbot browser storage session clear`
- `moltbot browser set offline on`
- `moltbot browser set headers --json '{"X-Debug":"1"}'`
- `moltbot browser set credentials user pass`
- `moltbot browser set credentials --clear`
- `moltbot browser set geo 37.7749 -122.4194 --origin "https://example.com"`
- `moltbot browser set geo --clear`
- `moltbot browser set media dark`
- `moltbot browser set timezone America/New_York`
- `moltbot browser set locale en-US`
- `moltbot browser set device "iPhone 14"`

说明：
- `upload` 和 `dialog` 是**装备**调用；在触发选择器/对话框的点击/按下之前运行它们。
- `upload` 也可以通过 `--input-ref` 或 `--element` 直接设置文件输入。
- `snapshot`：
  - `--format ai`（安装 Playwright 时默认）：返回带有数字引用的 AI 快照（`aria-ref="<n>"`）。
  - `--format aria`：返回可访问性树（无引用；仅检查）。
  - `--efficient`（或 `--mode efficient`）：紧凑的角色快照预设（交互式 + 紧凑 + 深度 + 较低 maxChars）。
  - 配置默认值（仅工具/CLI）：设置 `browser.snapshotDefaults.mode: "efficient"` 以在调用者未传递模式时使用高效快照（请参阅[网关配置](/gateway/configuration#browser-clawd-managed-browser)）。
  - 角色快照选项（`--interactive`、`--compact`、`--depth`、`--selector`）强制使用基于引用的快照，如 `ref=e12`。
  - `--frame "<iframe selector>"` 将角色快照范围限制为 iframe（与角色引用如 `e12` 配对）。
  - `--interactive` 输出一个扁平的、易于选择的交互元素列表（最适合驱动动作）。
  - `--labels` 添加带有覆盖引用标签的仅视口屏幕截图（打印 `MEDIA:<path>`）。
- `click`/`type`/等需要来自 `snapshot` 的 `ref`（数字 `12` 或角色引用 `e12`）。
  动作不支持 CSS 选择器。

## 快照和引用

Moltbot 支持两种"快照"样式：

- **AI 快照（数字引用）**：`moltbot browser snapshot`（默认；`--format ai`）
  - 输出：包含数字引用的文本快照。
  - 动作：`moltbot browser click 12`、`moltbot browser type 23 "hello"`。
  - 内部，引用通过 Playwright 的 `aria-ref` 解析。

- **角色快照（角色引用如 `e12`）**：`moltbot browser snapshot --interactive`（或 `--compact`、`--depth`、`--selector`、`--frame`）
  - 输出：基于角色的列表/树，带有 `[ref=e12]`（和可选的 `[nth=1]`）。
  - 动作：`moltbot browser click e12`、`moltbot browser highlight e12`。
  - 内部，引用通过 `getByRole(...)` 解析（加上 `nth()` 用于重复项）。
  - 添加 `--labels` 以包含带有覆盖 `e12` 标签的视口屏幕截图。

引用行为：
- 引用在导航之间**不稳定**；如果失败，重新运行 `snapshot` 并使用新引用。
- 如果角色快照是使用 `--frame` 拍摄的，角色引用范围限定为该 iframe，直到下一个角色快照。

## 等待增强功能

您可以等待的不仅仅是时间/文本：

- 等待 URL（Playwright 支持的通配符）：
  - `moltbot browser wait --url "**/dash"`
- 等待加载状态：
  - `moltbot browser wait --load networkidle`
- 等待 JS 谓词：
  - `moltbot browser wait --fn "window.ready===true"`
- 等待选择器变为可见：
  - `moltbot browser wait "#main"`

这些可以组合：

```bash
moltbot browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## 调试工作流程

当操作失败时（例如"不可见"、"严格模式违规"、"覆盖"）：

1. `moltbot browser snapshot --interactive`
2. 使用 `click <ref>` / `type <ref>`（在交互模式下优先使用角色引用）
3. 如果仍然失败：`moltbot browser highlight <ref>` 查看 Playwright 的目标
4. 如果页面行为异常：
   - `moltbot browser errors --clear`
   - `moltbot browser requests --filter api --clear`
5. 对于深度调试：记录跟踪：
   - `moltbot browser trace start`
   - 重现问题
   - `moltbot browser trace stop`（打印 `TRACE:<path>`）

## JSON 输出

`--json` 用于脚本和结构化工具。

示例：

```bash
moltbot browser status --json
moltbot browser snapshot --interactive --json
moltbot browser requests --filter api --json
moltbot browser cookies --json
```

JSON 中的角色快照包括 `refs` 加上一个小的 `stats` 块（行/字符/引用/交互），以便工具可以推断载荷大小和密度。

## 状态和环境控制

这些对于"使站点行为像 X"的工作流程很有用：

- Cookies：`cookies`、`cookies set`、`cookies clear`
- 存储：`storage local|session get|set|clear`
- 离线：`set offline on|off`
- 标题：`set headers --json '{"X-Debug":"1"}'`（或 `--clear`）
- HTTP basic 身份验证：`set credentials user pass`（或 `--clear`）
- 地理位置：`set geo <lat> <lon> --origin "https://example.com"`（或 `--clear`）
- 媒体：`set media dark|light|no-preference|none`
- 时区/语言环境：`set timezone ...`、`set locale ...`
- 设备/视口：
  - `set device "iPhone 14"`（Playwright 设备预设）
  - `set viewport 1280 720`

## 安全性和隐私

- clawd 浏览器配置文件可能包含已登录的会话；将其视为敏感信息。
- `browser act kind=evaluate` / `moltbot browser evaluate` 和 `wait --fn` 在页面上下文中执行任意 JavaScript。提示注入可以引导此操作。如果不需要，请通过 `browser.evaluateEnabled=false` 禁用它。
- 有关登录和反机器人说明（X/Twitter 等），请参阅[浏览器登录 + X/Twitter 发布](/tools/browser-login)。
- 将网关/节点主机保持私密（仅回环或 tailnet）。
- 远程 CDP 端点功能强大；通过隧道保护它们。

## 故障排除

有关 Linux 特定问题（尤其是 snap Chromium），请参阅[浏览器故障排除](/tools/browser-linux-troubleshooting)。

## 代理工具 + 控制工作原理

代理获得**一个工具**用于浏览器自动化：
- `browser` — 状态/启动/停止/标签页/打开/聚焦/关闭/快照/截图/导航/动作

它的映射方式：
- `browser snapshot` 返回稳定的 UI 树（AI 或 ARIA）。
- `browser act` 使用快照 `ref` ID 来点击/输入/拖动/选择。
- `browser screenshot` 捕获像素（整页或元素）。
- `browser` 接受：
  - `profile` 以选择命名的浏览器配置文件（clawd、chrome 或远程 CDP）。
  - `target`（`sandbox` | `host` | `node`）以选择浏览器的位置。
  - 在沙箱会话中，`target: "host"` 需要 `agents.defaults.sandbox.browser.allowHostControl=true`。
  - 如果省略 `target`：沙箱会话默认为 `sandbox`，非沙箱会话默认为 `host`。
  - 如果连接了支持浏览器的节点，工具可能会自动路由到它，除非您固定 `target="host"` 或 `target="node"`。

这使代理保持确定性，并避免了脆弱的选择器。
