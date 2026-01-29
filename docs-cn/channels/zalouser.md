---
summary: "通过 zca-cli (QR 登录)的 Zalo 个人账户支持、功能和配置"
read_when:
  - 为 Moltbot 设置 Zalo Personal
  - 调试 Zalo Personal 登录或消息流
---
# Zalo Personal (非官方)

状态: 实验性。此集成通过 `zca-cli` 自动化**个人 Zalo 账户**。

> **警告:** 这是一个非官方集成，可能导致账户暂停/封禁。使用风险自负。

## 需要插件
Zalo Personal 作为插件提供，不包含在核心安装中。
- 通过 CLI 安装: `moltbot plugins install @moltbot/zalouser`
- 或从源检出: `moltbot plugins install ./extensions/zalouser`
- 详情: [插件](/plugin)

## 前提条件: zca-cli
网关机器必须在 `PATH` 中有 `zca` 二进制文件可用。

- 验证: `zca --version`
- 如果缺失，安装 zca-cli(参见 `extensions/zalouser/README.md` 或上游 zca-cli 文档)。

## 快速设置(初学者)
1) 安装插件(见上文)。
2) 登录(QR，在网关机器上):
   - `moltbot channels login --channel zalouser`
   - 在终端中使用 Zalo 移动应用扫描 QR 代码。
3) 启用频道:

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

4) 重启网关(或完成入职)。
5) 私信访问默认为配对；首次联系时批准配对代码。

## 它是什么
- 使用 `zca listen` 接收入站消息。
- 使用 `zca msg ...` 发送回复(文本/媒体/链接)。
- 设计用于 Zalo Bot API 不可用的"个人账户"用例。

## 命名
频道 id 是 `zalouser`，以明确这自动化**个人 Zalo 用户账户**(非官方)。我们保留 `zalo` 用于潜在的未来官方 Zalo API 集成。

## 查找 ID(目录)
使用目录 CLI 发现对等方/群组及其 ID:

```bash
moltbot directory self --channel zalouser
moltbot directory peers list --channel zalouser --query "name"
moltbot directory groups list --channel zalouser --query "work"
```

## 限制
- 出站文本分块为约 2000 个字符(Zalo 客户端限制)。
- 默认阻止流式传输。

## 访问控制(私信)
`channels.zalouser.dmPolicy` 支持: `pairing | allowlist | open | disabled`(默认: `pairing`)。
`channels.zalouser.allowFrom` 接受用户 ID 或名称。向导在可用时通过 `zca friend find` 将名称解析为 ID。

通过以下方式批准:
- `moltbot pairing list zalouser`
- `moltbot pairing approve zalouser <code>`

## 群组访问(可选)
- 默认: `channels.zalouser.groupPolicy = "open"`(允许群组)。使用 `channels.defaults.groupPolicy` 在未设置时覆盖默认值。
- 使用允许列表限制:
  - `channels.zalouser.groupPolicy = "allowlist"`
  - `channels.zalouser.groups`(键是群组 ID 或名称)
- 阻止所有群组: `channels.zalouser.groupPolicy = "disabled"`。
- 配置向导可以提示群组允许列表。
- 启动时，Moltbot 将允许列表中的群组/用户名称解析为 ID 并记录映射；未解析的条目保持输入状态。

示例:
```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

## 多账户
账户映射到 zca 配置文件。示例:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

## 故障排除

**找不到 `zca`:**
- 安装 zca-cli 并确保它在网关进程的 `PATH` 上。

**登录不持久:**
- `moltbot channels status --probe`
- 重新登录: `moltbot channels logout --channel zalouser && moltbot channels login --channel zalouser`
