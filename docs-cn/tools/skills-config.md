---
summary: "技能配置架构和示例"
read_when:
  - 添加或修改技能配置
  - 调整捆绑允许列表或安装行为
---
# 技能配置

所有与技能相关的配置都位于 `~/.clawdbot/moltbot.json` 中的 `skills` 下。

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun（网关运行时仍然是 Node；不推荐 bun）
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

## 字段

- `allowBundled`：可选的**仅限捆绑**技能的允许列表。设置后，仅列表中的捆绑技能符合条件（托管/工作区技能不受影响）。
- `load.extraDirs`：要扫描的其他技能目录（最低优先级）。
- `load.watch`：监视技能文件夹并刷新技能快照（默认：true）。
- `load.watchDebounceMs`：技能监视器事件的去抖动时间（毫秒）（默认：250）。
- `install.preferBrew`：可用时优先使用 brew 安装程序（默认：true）。
- `install.nodeManager`：node 安装程序首选项（`npm` | `pnpm` | `yarn` | `bun`，默认：npm）。
  这仅影响**技能安装**；网关运行时应该仍然是 Node（不建议将 Bun 用于 WhatsApp/Telegram）。
- `entries.<skillKey>`：每技能覆盖。

每技能字段：
- `enabled`：设置为 `false` 以禁用技能，即使它已捆绑/安装。
- `env`：为代理运行注入的环境变量（仅在尚未设置时）。
- `apiKey`：为声明主要环境变量的技能提供的可选便利。

## 说明

- `entries` 下的键默认映射到技能名称。如果技能定义 `metadata.moltbot.skillKey`，请改为使用该键。
- 当启用监视器时，对技能的更改在下一个代理轮次时被拾取。

### 沙箱技能 + 环境变量

当会话**处于沙箱状态**时，技能进程在 Docker 内运行。沙箱**不会**继承主机 `process.env`。

使用以下之一：
- `agents.defaults.sandbox.docker.env`（或每代理 `agents.list[].sandbox.docker.env`）
- 将环境变量烘焙到您的自定义沙箱镜像中

全局 `env` 和 `skills.entries.<skill>.env/apiKey` 仅适用于**主机**运行。
