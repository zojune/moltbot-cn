---
summary: "macOS UI 自动化的 PeekabooBridge 集成"
read_when:
  - 在 Moltbot.app 中托管 PeekabooBridge
  - 通过 Swift Package Manager 集成 Peekaboo
  - 更改 PeekabooBridge 协议/路径
---
# Peekaboo Bridge（macOS UI 自动化）

Moltbot 可以托管 **PeekabooBridge** 作为本地、感知权限的 UI 自动化
代理。这使 `peekaboo` CLI 可以驱动 UI 自动化，同时重用
macOS 应用的 TCC 权限。

## 这是什么（以及不是什么）

- **主机：** Moltbot.app 可以充当 PeekabooBridge 主机。
- **客户端：** 使用 `peekaboo` CLI（没有单独的 `moltbot ui ...` 表面）。
- **UI：** 视觉叠加层保留在 Peekaboo.app 中；Moltbot 是一个瘦代理主机。

## 启用网桥

在 macOS 应用中：
- 设置 → **启用 Peekaboo Bridge**

启用后，Moltbot 启动本地 UNIX 套接字服务器。如果禁用，主机
将停止，`peekaboo` 将回退到其他可用主机。

## 客户端发现顺序

Peekaboo 客户端通常按以下顺序尝试主机：

1. Peekaboo.app（完整 UX）
2. Claude.app（如果已安装）
3. Moltbot.app（瘦代理）

使用 `peekaboo bridge status --verbose` 查看哪个主机处于活动状态以及哪个
套接字路径正在使用。您可以使用以下方式覆盖：

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## 安全性和权限

- 网桥验证**调用者代码签名**；强制执行 TeamID 的允许列表
  (Peekaboo 主机 TeamID + Moltbot 应用 TeamID)。
- 请求在大约 10 秒后超时。
- 如果缺少所需权限，网桥会返回清晰的错误消息
  而不是启动系统设置。

## 快照行为（自动化）

快照存储在内存中并在短时间窗口后自动过期。
如果您需要更长的保留时间，请从客户端重新捕获。

## 故障排除

- 如果 `peekaboo` 报告"网桥客户端未获授权"，请确保客户端
  已正确签名或在**仅调试**模式下使用 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  运行主机。
- 如果找不到主机，请打开主机应用之一（Peekaboo.app 或 Moltbot.app）
  并确认已授予权限。
