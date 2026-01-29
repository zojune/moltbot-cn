---
summary: "`moltbot agents` CLI 参考(列表/添加/删除/设置身份)"
read_when:
  - 您需要多个隔离的 agent(工作区 + 路由 + 认证)
---

# `moltbot agents`

管理隔离的 agent(工作区 + 认证 + 路由)。

相关:
- 多 agent 路由: [Multi-Agent Routing](/concepts/multi-agent)
- Agent 工作区: [Agent workspace](/concepts/agent-workspace)

## 示例

```bash
moltbot agents list
moltbot agents add work --workspace ~/clawd-work
moltbot agents set-identity --workspace ~/clawd --from-identity
moltbot agents set-identity --agent main --avatar avatars/clawd.png
moltbot agents delete work
```

## 身份文件

每个 agent 工作区可以在工作区根目录包含一个 `IDENTITY.md` 文件:
- 示例路径: `~/clawd/IDENTITY.md`
- `set-identity --from-identity` 从工作区根目录读取(或显式的 `--identity-file`)

头像路径相对于工作区根目录解析。

## 设置身份

`set-identity` 将字段写入 `agents.list[].identity`:
- `name`
- `theme`
- `emoji`
- `avatar`(工作区相对路径、http(s) URL 或 data URI)

从 `IDENTITY.md` 加载:

```bash
moltbot agents set-identity --workspace ~/clawd --from-identity
```

显式覆盖字段:

```bash
moltbot agents set-identity --agent main --name "Clawd" --emoji "🦞" --avatar avatars/clawd.png
```

配置示例:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Clawd",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/clawd.png"
        }
      }
    ]
  }
}
```
