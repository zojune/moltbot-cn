---
summary: "平台支持概述（网关 + 配套应用）"
read_when:
  - 寻找操作系统支持或安装路径
  - 决定在哪里运行网关
---
# 平台

Moltbot 核心用 TypeScript 编写。**Node 是推荐的运行时**。
不建议将 Bun 用于网关（WhatsApp/Telegram 错误）。

配套应用适用于 macOS（菜单栏应用）和移动节点（iOS/Android）。Windows 和
Linux 配套应用已计划中，但网关今天已完全支持。
原生 Windows 配套应用也已计划；推荐通过 WSL2 使用网关。

## 选择您的操作系统

- macOS：[macOS](/platforms/macos)
- iOS：[iOS](/platforms/ios)
- Android：[Android](/platforms/android)
- Windows：[Windows](/platforms/windows)
- Linux：[Linux](/platforms/linux)

## VPS 和托管

- VPS 中心：[VPS 托管](/vps)
- Fly.io：[Fly.io](/platforms/fly)
- Hetzner (Docker)：[Hetzner](/platforms/hetzner)
- GCP (Compute Engine)：[GCP](/platforms/gcp)
- exe.dev (VM + HTTPS 代理)：[exe.dev](/platforms/exe-dev)

## 常用链接

- 安装指南：[入门](/start/getting-started)
- 网关操作手册：[网关](/gateway)
- 网关配置：[配置](/gateway/configuration)
- 服务状态：`moltbot gateway status`

## 网关服务安装（CLI）

使用以下之一（全部支持）：

- 向导（推荐）：`moltbot onboard --install-daemon`
- 直接：`moltbot gateway install`
- 配置流程：`moltbot configure` → 选择 **网关服务**
- 修复/迁移：`moltbot doctor`（提供安装或修复服务）

服务目标取决于操作系统：
- macOS：LaunchAgent（`bot.molt.gateway` 或 `bot.molt.<profile>`；传统 `com.clawdbot.*`）
- Linux/WSL2：systemd 用户服务（`moltbot-gateway[-<profile>].service`）
