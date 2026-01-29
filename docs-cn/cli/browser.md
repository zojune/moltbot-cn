---
summary: "`moltbot browser` CLI 参考(配置文件、标签、操作、扩展中继)"
read_when:
  - 您使用 `moltbot browser` 并希望获得常见任务的示例
  - 您希望通过 node 主机控制在另一台机器上运行的浏览器
  - 您想使用 Chrome 扩展中继(通过工具栏按钮附加/分离)
---

# `moltbot browser`

管理 Moltbot 的浏览器控制服务器并运行浏览器操作(标签、快照、截图、导航、点击、输入)。

相关:
- 浏览器工具 + API: [Browser tool](/tools/browser)
- Chrome 扩展中继: [Chrome extension](/tools/chrome-extension)

## 常用标志

- `--url <gatewayWsUrl>`: Gateway WebSocket URL(默认为配置)。
- `--token <token>`: Gateway 令牌(如果需要)。
- `--timeout <ms>`: 请求超时(毫秒)。
- `--browser-profile <name>`: 选择浏览器配置文件(默认来自配置)。
- `--json`: 机器可读的输出(在支持的地方)。

## 快速开始(本地)

```bash
moltbot browser --browser-profile chrome tabs
moltbot browser --browser-profile clawd start
moltbot browser --browser-profile clawd open https://example.com
moltbot browser --browser-profile clawd snapshot
```

## 配置文件

配置文件是命名的浏览器路由配置。实际上:
- `clawd`: 启动/附加到专用的 Moltbot 管理的 Chrome 实例(隔离的用户数据目录)。
- `chrome`: 通过 Chrome 扩展中继控制您现有的 Chrome 标签。

```bash
moltbot browser profiles
moltbot browser create-profile --name work --color "#FF5A36"
moltbot browser delete-profile --name work
```

使用特定的配置文件:

```bash
moltbot browser --browser-profile work tabs
```

## 标签

```bash
moltbot browser tabs
moltbot browser open https://docs.molt.bot
moltbot browser focus <targetId>
moltbot browser close <targetId>
```

## 快照 / 截图 / 操作

快照:

```bash
moltbot browser snapshot
```

截图:

```bash
moltbot browser screenshot
```

导航/点击/输入(基于 ref 的 UI 自动化):

```bash
moltbot browser navigate https://example.com
moltbot browser click <ref>
moltbot browser type <ref> "hello"
```

## Chrome 扩展中继(通过工具栏按钮附加)

此模式允许 agent 控制您手动附加的现有 Chrome 标签(它不会自动附加)。

将解压的扩展安装到稳定路径:

```bash
moltbot browser extension install
moltbot browser extension path
```

然后 Chrome → `chrome://extensions` → 启用"开发者模式" → "加载解压的扩展" → 选择打印的文件夹。

完整指南: [Chrome extension](/tools/chrome-extension)

## 远程浏览器控制(node 主机代理)

如果 Gateway 在与浏览器不同的机器上运行,请在拥有 Chrome/Brave/Edge/Chromium 的机器上运行 **node 主机**。Gateway 将浏览器操作代理到该 node(不需要单独的浏览器控制服务器)。

使用 `gateway.nodes.browser.mode` 控制自动路由,使用 `gateway.nodes.browser.node` 在连接多个 node 时固定特定的 node。

安全 + 远程设置: [Browser tool](/tools/browser)、[Remote access](/gateway/remote)、[Tailscale](/gateway/tailscale)、[Security](/gateway/security)
