---
summary: "Zalo 个人版插件：通过 zca-cli 进行二维码登录和消息传递（插件安装 + 频道配置 + CLI + 工具）"
read_when:
  - 您想在 Moltbot 中获得 Zalo 个人版（非官方）支持
  - 您正在配置或开发 zalouser 插件
---

# Zalo 个人版（插件）

通过插件为 Moltbot 提供 Zalo 个人版支持，使用 `zca-cli` 来自动化普通的 Zalo 用户账户。

> **警告：** 非官方自动化可能会导致账户暂停/封禁。使用风险自负。

## 命名

频道 id 是 `zalouser`，以明确这是自动化**个人 Zalo 用户账户**（非官方）。我们保留 `zalo` 用于将来可能的官方 Zalo API 集成。

## 运行位置
此插件在**网关进程内**运行。

如果您使用远程网关，请在**运行网关的机器上**安装/配置它，然后重启网关。

## 安装

### 选项 A：从 npm 安装

```bash
moltbot plugins install @moltbot/zalouser
```

之后重启网关。

### 选项 B：从本地文件夹安装（开发）

```bash
moltbot plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

之后重启网关。

## 前提条件：zca-cli
网关机器必须在 `PATH` 上有 `zca`：

```bash
zca --version
```

## 配置
频道配置位于 `channels.zalouser` 下（而非 `plugins.entries.*`）：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```

## CLI

```bash
moltbot channels login --channel zalouser
moltbot channels logout --channel zalouser
moltbot channels status --probe
moltbot message send --channel zalouser --target <threadId> --message "Hello from Moltbot"
moltbot directory peers list --channel zalouser --query "name"
```

## 代理工具
工具名称：`zalouser`

操作：`send`、`image`、`link`、`friends`、`groups`、`me`、`status`
