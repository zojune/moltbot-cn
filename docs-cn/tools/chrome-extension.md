---
summary: "Chrome 扩展：让 Moltbot 驱动您现有的 Chrome 标签页"
read_when:
  - 您希望代理驱动现有的 Chrome 标签页（工具栏按钮）
  - 您需要远程网关 + 通过 Tailscale 进行本地浏览器自动化
  - 您想了解浏览器接管的安全影响
---

# Chrome 扩展（浏览器中继）

Moltbot Chrome 扩展允许代理控制您的**现有 Chrome 标签页**（您的正常 Chrome 窗口），而不是启动单独的 clawd 托管 Chrome 配置文件。

附加/分离通过**单个 Chrome 工具栏按钮**进行。

## 它是什么（概念）

有三个部分：
- **浏览器控制服务**（网关或节点）：代理/工具调用的 API（通过网关）
- **本地中继服务器**（回环 CDP）：在控制服务器和扩展之间架桥（`http://127.0.0.1:18792` 默认）
- **Chrome MV3 扩展**：使用 `chrome.debugger` 附加到活动标签页并将 CDP 消息通过管道传输到中继

然后 Moltbot 通过正常的 `browser` 工具界面（选择正确的配置文件）控制附加的标签页。

## 安装/加载（未打包）

1) 将扩展安装到稳定的本地路径：

```bash
moltbot browser extension install
```

2) 打印已安装的扩展目录路径：

```bash
moltbot browser extension path
```

3) Chrome → `chrome://extensions`
- 启用"开发者模式"
- "加载未打包" → 选择上面打印的目录

4) 固定扩展。

## 更新（无需构建步骤）

扩展作为静态文件随 Moltbot 版本（npm 软件包）一起提供。没有单独的"构建"步骤。

升级 Moltbot 后：
- 重新运行 `moltbot browser extension install` 以刷新 Moltbot 状态目录下的已安装文件。
- Chrome → `chrome://extensions` → 点击扩展上的"重新加载"。

## 使用它（无需额外配置）

Moltbot 随附一个名为 `chrome` 的内置浏览器配置文件，用于默认端口上的扩展中继。

使用它：
- CLI：`moltbot browser --browser-profile chrome tabs`
- 代理工具：`browser` 与 `profile="chrome"`

如果您想要不同的名称或不同的中继端口，请创建自己的配置文件：

```bash
moltbot browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

## 附加/分离（工具栏按钮）

- 打开您希望 Moltbot 控制的标签页。
- 点击扩展图标。
  - 徽章在附加时显示 `ON`。
- 再次点击以分离。

## 它控制哪个标签页？

- 它**不会**自动控制"您正在查看的任何标签页"。
- 它**仅控制您通过点击工具栏按钮明确附加的标签页**。
- 要切换：打开其他标签页并在那里点击扩展图标。

## 徽章 + 常见错误

- `ON`：已附加；Moltbot 可以驱动该标签页。
- `…`：正在连接到本地中继。
- `!`：中继不可达（最常见：浏览器中继服务器未在此机器上运行）。

如果您看到 `!`：
- 确保网关在本地运行（默认设置），或者如果网关在其他地方运行，请在此机器上运行节点主机。
- 打开扩展选项页面；它显示中继是否可达。

## 远程网关（使用节点主机）

### 本地网关（与 Chrome 在同一台机器上） - 通常**无需额外步骤**

如果网关在与 Chrome 相同的机器上运行，它会在回环上启动浏览器控制服务并自动启动中继服务器。扩展与本地中继通信；CLI/工具调用转到网关。

### 远程网关（网关在其他地方运行） - **运行节点主机**

如果您的网关在另一台机器上运行，请在运行 Chrome 的机器上启动节点主机。网关将浏览器动作代理到该节点；扩展 + 中继保持在浏览器机器的本地。

如果连接了多个节点，请使用 `gateway.nodes.browser.node` 固定一个或设置 `gateway.nodes.browser.mode`。

## 沙箱（工具容器）

如果您的代理会话处于沙箱状态（`agents.defaults.sandbox.mode != "off"`），`browser` 工具可能会受到限制：

- 默认情况下，沙箱会话通常定向到**沙箱浏览器**（`target="sandbox"`），而不是您的主机 Chrome。
- Chrome 扩展中继接管需要控制**主机**浏览器控制服务器。

选项：
- 最简单：从**非沙箱**会话/代理使用扩展。
- 或者允许沙箱会话的主机浏览器控制：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

然后确保工具未被工具策略拒绝，并且（如果需要）使用 `target="host"` 调用 `browser`。

调试：`moltbot sandbox explain`

## 远程访问提示

- 将网关和节点主机保持在同一 tailnet 上；避免向 LAN 或公共互联网暴露中继端口。
- 有意配对节点；如果您不想要远程控制，请禁用浏览器代理路由（`gateway.nodes.browser.mode="off"`）。

## "扩展路径"的工作原理

`moltbot browser extension path` 打印包含扩展文件的**已安装**磁盘目录。

CLI 故意**不**打印 `node_modules` 路径。始终先运行 `moltbot browser extension install` 以将扩展复制到 Moltbot 状态目录下的稳定位置。

如果您移动或删除该安装目录，Chrome 将标记扩展为已损坏，直到您从有效路径重新加载它。

## 安全影响（请阅读）

这很强大且有风险。将其视为给模型"控制您的浏览器"。

- 扩展使用 Chrome 的调试器 API（`chrome.debugger`）。附加时，模型可以：
  - 在该标签页中点击/输入/导航
  - 阅读页面内容
  - 访问标签页的已登录会话可以访问的任何内容
- **这与专用的 clawd 托管配置文件不同**，它不是隔离的。
  - 如果您附加到日常驱动程序配置文件/标签页，您将授予对该帐户状态的访问权限。

建议：
- 为扩展中继使用专用的 Chrome 配置文件（与个人浏览分开）。
- 将网关和任何节点主机保持仅 tailnet；依赖网关身份验证 + 节点配对。
- 避免通过 LAN（`0.0.0.0`）暴露中继端口并避免 Funnel（公共）。

相关：
- 浏览器工具概述：[浏览器](/tools/browser)
- 安全审计：[安全性](/gateway/security)
- Tailscale 设置：[Tailscale](/gateway/tailscale)
