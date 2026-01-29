---
summary: "Moltbot.app 连接到远程 gateway 的 SSH 隧道设置"
read_when: "通过 SSH 将 macOS 应用连接到远程 gateway"
---

# 使用 Remote Gateway 运行 Moltbot.app

Moltbot.app 使用 SSH 隧道连接到远程 gateway。本指南向您展示如何设置它。

## 概述

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端机器                             │
│                                                              │
│  Moltbot.app ──► ws://127.0.0.1:18789 (本地端口)            │
│                     │                                        │
│                     ▼                                        │
│  SSH 隧道 ───────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         远程机器                              │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 快速设置

### 步骤 1：添加 SSH 配置

编辑 `~/.ssh/config` 并添加：

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # 例如，172.27.187.184
    User <REMOTE_USER>            # 例如，jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

将 `<REMOTE_IP>` 和 `<REMOTE_USER>` 替换为您的值。

### 步骤 2：复制 SSH 密钥

将您的公钥复制到远程机器（输入一次密码）：

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### 步骤 3：设置 Gateway Token

```bash
launchctl setenv CLAWDBOT_GATEWAY_TOKEN "<your-token>"
```

### 步骤 4：启动 SSH 隧道

```bash
ssh -N remote-gateway &
```

### 步骤 5：重启 Moltbot.app

```bash
# 退出 Moltbot.app（⌘Q），然后重新打开：
open /path/to/Moltbot.app
```

应用程序现在将通过 SSH 隧道连接到远程 gateway。

---

## 登录时自动启动隧道

要让 SSH 隧道在您登录时自动启动，请创建一个 Launch Agent。

### 创建 PLIST 文件

将其保存为 `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### 加载 Launch Agent

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

隧道现在将：
- 在您登录时自动启动
- 在崩溃时重新启动
- 在后台保持运行

遗留说明：如果存在，请删除任何剩余的 `com.clawdbot.ssh-tunnel` LaunchAgent。

---

## 故障排除

**检查隧道是否正在运行：**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**重启隧道：**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**停止隧道：**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

---

## 工作原理

| 组件 | 它的作用 |
|-----------|--------------|
| `LocalForward 18789 127.0.0.1:18789` | 将本地端口 18789 转发到远程端口 18789 |
| `ssh -N` | SSH 不执行远程命令（仅端口转发） |
| `KeepAlive` | 隧道崩溃时自动重新启动 |
| `RunAtLoad` | 代理加载时启动隧道 |

Moltbot.app 连接到客户端机器上的 `ws://127.0.0.1:18789`。SSH 隧道将该连接转发到运行 Gateway 的远程机器上的端口 18789。
