---
summary: "`moltbot directory` CLI 参考(自己、对等点、组)"
read_when:
  - 您想查找频道的联系人/组/自己的 id
  - 您正在开发频道目录适配器
---

# `moltbot directory`

支持目录的频道的目录查找(联系人/对等点、组和"自己")。

## 常用标志
- `--channel <name>`: 频道 id/别名(配置多个频道时需要;仅配置一个时自动)
- `--account <id>`: 账户 id(默认:频道默认值)
- `--json`: 输出 JSON

## 注意事项
- `directory` 旨在帮助您找到可以粘贴到其他命令中的 ID(特别是 `moltbot message send --target ...`)。
- 对于许多频道,结果是配置支持的(允许列表/配置的组),而不是实时的提供商目录。
- 默认输出是 `id`(有时还有 `name`),用制表符分隔;使用 `--json` 进行脚本编写。

## 将结果与 `message send` 一起使用

```bash
moltbot directory peers list --channel slack --query "U0"
moltbot message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID 格式(按频道)

- WhatsApp: `+15551234567`(DM)、`1234567890-1234567890@g.us`(组)
- Telegram: `@username` 或数字聊天 id;组是数字 id
- Slack: `user:U…` 和 `channel:C…`
- Discord: `user:<id>` 和 `channel:<id>`
- Matrix(插件): `user:@user:server`、`room:!roomId:server` 或 `#alias:server`
- Microsoft Teams(插件): `user:<id>` 和 `conversation:<id>`
- Zalo(插件): 用户 id(Bot API)
- Zalo Personal / `zalouser`(插件): 来自 `zca` 的线程 id(DM/组)(`me`、`friend list`、`group list`)

## 自己("我")

```bash
moltbot directory self --channel zalouser
```

## 对等点(联系人/用户)

```bash
moltbot directory peers list --channel zalouser
moltbot directory peers list --channel zalouser --query "name"
moltbot directory peers list --channel zalouser --limit 50
```

## 组

```bash
moltbot directory groups list --channel zalouser
moltbot directory groups list --channel zalouser --query "work"
moltbot directory groups members --channel zalouser --group-id <id>
```
