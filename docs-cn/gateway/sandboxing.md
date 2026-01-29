---
summary: "Moltbot 沙箱化的工作原理：模式、作用域、工作区访问和镜像"
title: 沙箱化
read_when: "您想要沙箱化的专门说明或需要调整 agents.defaults.sandbox。"
status: active
---

# 沙箱化

Moltbot 可以**在 Docker 容器内运行工具**以减少攻击面。
这是**可选的**，由配置控制（`agents.defaults.sandbox` 或
`agents.list[].sandbox`）。如果沙箱化关闭，工具在主机上运行。
Gateway 保持在主机上；工具执行在启用时在隔离的沙箱中运行。

这不是一个完美的安全边界，但当模型做愚蠢的事情时，它实质性地限制了文件系统
和进程访问。

## 什么被沙箱化
- 工具执行（`exec`、`read`、`write`、`edit`、`apply_patch`、`process` 等）。
- 可选的沙箱化浏览器（`agents.defaults.sandbox.browser`）。
  - 默认情况下，当浏览器工具需要时，沙箱浏览器会自动启动（确保 CDP 可达）。
    通过 `agents.defaults.sandbox.browser.autoStart` 和 `agents.defaults.sandbox.browser.autoStartTimeoutMs` 配置。
  - `agents.defaults.sandbox.browser.allowHostControl` 允许沙箱化会话显式定位主机浏览器。
  - 可选的允许列表控制 `target: "custom"`：`allowedControlUrls`、`allowedControlHosts`、`allowedControlPorts`。

未沙箱化：
- Gateway 进程本身。
- 任何显式允许在主机上运行的工具（例如 `tools.elevated`）。
  - **提升权限 exec 在主机上运行并绕过沙箱化。**
  - 如果沙箱化关闭，`tools.elevated` 不会改变执行（已经在主机上）。请参阅[提升权限模式](/tools/elevated)。

## 模式
`agents.defaults.sandbox.mode` 控制**何时**使用沙箱化：
- `"off"`：没有沙箱化。
- `"non-main"`：仅沙箱化**非 main** 会话（如果您希望在主机上进行正常聊天，则为默认）。
- `"all"`：每个会话都在沙箱中运行。
注意：`"non-main"` 基于 `session.mainKey`（默认 `"main"`），而不是 agent id。
群组/通道会话使用自己的键，因此它们算作非 main 并将被沙箱化。

## 作用域
`agents.defaults.sandbox.scope` 控制**创建多少个容器**：
- `"session"`（默认）：每个会话一个容器。
- `"agent"`：每个 agent 一个容器。
- `"shared"`：所有沙箱化会话共享一个容器。

## 工作区访问
`agents.defaults.sandbox.workspaceAccess` 控制**沙箱可以看到什么**：
- `"none"`（默认）：工具在 `~/.clawdbot/sandboxes` 下的沙箱工作区。
- `"ro"`：以只读方式挂载 agent 工作区到 `/agent`（禁用 `write`/`edit`/`apply_patch`）。
- `"rw"`：以读写方式挂载 agent 工作区到 `/workspace`。

入站媒体被复制到活动沙箱工作区（`media/inbound/*`）。
技能注意：`read` 工具是沙箱根目录的。使用 `workspaceAccess: "none"`，
Moltbot 将符合条件的技能镜像到沙箱工作区（`.../skills`），以便
它们可以被读取。使用 `"rw"`，工作区技能可从
`/workspace/skills` 读取。

## 自定义绑定挂载
`agents.defaults.sandbox.docker.binds` 将额外的主机目录挂载到容器中。
格式：`host:container:mode`（例如，`"/home/user/source:/source:rw"`）。

全局和每个 agent 的绑定是**合并的**（不替换）。在 `scope: "shared"` 下，每个 agent 的绑定被忽略。

示例（只读源 + docker socket）：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

安全注意事项：
- 绑定绕过沙箱文件系统：它们以您设置的任何模式（`:ro` 或 `:rw`）暴露主机路径。
- 敏感挂载（例如 `docker.sock`、机密、SSH 密钥）应该是 `:ro`，除非绝对必要。
- 如果您只需要对工作区的读访问权限，请结合 `workspaceAccess: "ro"`；绑定模式保持独立。
- 请参阅[沙箱 vs 工具策略 vs 提升权限](/gateway/sandbox-vs-tool-policy-vs-elevated)以了解绑定如何与工具策略和提升权限 exec 交互。

## 镜像 + 设置
默认镜像：`moltbot-sandbox:bookworm-slim`

构建一次：
```bash
scripts/sandbox-setup.sh
```

注意：默认镜像**不**包括 Node。如果技能需要 Node（或其他运行时），请烘焙自定义镜像或通过
`sandbox.docker.setupCommand` 安装（需要网络出口 + 可写根 + root 用户）。

沙箱化浏览器镜像：
```bash
scripts/sandbox-browser-setup.sh
```

默认情况下，沙箱容器以**无网络**运行。
通过 `agents.defaults.sandbox.docker.network` 覆盖。

Docker 安装和容器化 gateway 位于这里：
[Docker](/install/docker)

## setupCommand（一次性容器设置）
`setupCommand` 在创建沙箱容器后运行**一次**（不是每次运行）。
它通过 `sh -lc` 在容器内执行。

路径：
- 全局：`agents.defaults.sandbox.docker.setupCommand`
- 每个 agent：`agents.list[].sandbox.docker.setupCommand`


常见陷阱：
- 默认 `docker.network` 是 `"none"`（无出口），因此包安装将失败。
- `readOnlyRoot: true` 防止写入；设置 `readOnlyRoot: false` 或烘焙自定义镜像。
- `user` 必须是 root 才能进行包安装（省略 `user` 或设置 `user: "0:0"`）。
- 沙箱 exec **不**继承主机 `process.env`。使用
  `agents.defaults.sandbox.docker.env`（或自定义镜像）作为技能 API 密钥。

## 工具策略 + 逃生舱
工具允许/拒绝策略在沙箱规则之前仍然适用。如果工具被全局或每个 agent 拒绝，
沙箱化不会恢复它。

`tools.elevated` 是一个显式的逃生舱，它在主机上运行 `exec`。
`/exec` 指令仅适用于已授权的发送者并每个会话持久化；要硬禁用
`exec`，请使用工具策略拒绝（请参阅[沙箱 vs 工具策略 vs 提升权限](/gateway/sandbox-vs-tool-policy-vs-elevated)）。

调试：
- 使用 `moltbot sandbox explain` 检查有效的沙箱模式、工具策略和修复配置键。
- 请参阅[沙箱 vs 工具策略 vs 提升权限](/gateway/sandbox-vs-tool-policy-vs-elevated)以了解"为什么这被阻止？"心智模型。
保持锁定。

## 多 agent 覆盖
每个 agent 可以覆盖沙箱 + 工具：
`agents.list[].sandbox` 和 `agents.list[].tools`（加上 `agents.list[].tools.sandbox.tools` 用于沙箱工具策略）。
请参阅[多 Agent 沙箱和工具](/multi-agent-sandbox-tools)以了解优先级。

## 最小启用示例
```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

## 相关文档
- [沙箱配置](/gateway/configuration#agentsdefaults-sandbox)
- [多 Agent 沙箱和工具](/multi-agent-sandbox-tools)
- [安全性](/gateway/security)
