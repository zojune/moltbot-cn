---
summary: "浏览器自动化手动登录 + X/Twitter 发布"
read_when:
  - 需要登录站点进行浏览器自动化
  - 想要向 X/Twitter 发布更新
---

# 浏览器登录 + X/Twitter 发布

## 手动登录（推荐）

当站点需要登录时，**在主机浏览器配置文件**（clawd 浏览器）中**手动登录**。

请**不要**给模型您的凭据。自动登录通常会触发反机器人防御并可能导致帐户被锁定。

返回主浏览器文档：[浏览器](/tools/browser)

## 使用哪个 Chrome 配置文件？

Moltbot 控制一个**专用 Chrome 配置文件**（名为 `clawd`，橙色色调 UI）。这与您的日常浏览器配置文件是分开的。

两种简单的访问方式：

1) **让代理为您打开浏览器**，然后自己登录。
2) **通过 CLI 打开**：

```bash
moltbot browser start
moltbot browser open https://x.com
```

如果您有多个配置文件，请传递 `--browser-profile <name>`（默认为 `clawd`）。

## X/Twitter：推荐流程

- **阅读/搜索/线程：**使用 **bird** CLI 技能（无浏览器，稳定）。
  - 仓库：https://github.com/steipete/bird
- **发布更新：**使用**主机浏览器**（手动登录）。

## 沙箱 + 主机浏览器访问

沙箱浏览器会话**更有可能**触发机器人检测。对于 X/Twitter（和其他严格站点），首选**主机浏览器**。

如果代理处于沙箱状态，浏览器工具默认使用沙箱。要允许主机控制：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

然后定向主机浏览器：

```bash
moltbot browser open https://x.com --browser-profile clawd --target host
```

或者为发布更新的代理禁用沙箱。
