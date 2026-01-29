---
summary: "技能：托管 vs 工作区、限制规则和配置/环境连接"
read_when:
  - 添加或修改技能
  - 更改技能限制或加载规则
---
# 技能（Moltbot）

Moltbot 使用 **[AgentSkills](https://agentskills.io)-兼容**技能文件夹来教代理如何使用工具。每个技能都是一个包含 `SKILL.md` 的目录，具有 YAML frontmatter 和说明。Moltbot 加载**捆绑技能**加上可选的本地覆盖，并根据环境、配置和二进制文件在加载时过滤它们。

## 位置和优先级

技能从**三个**地方加载：

1) **捆绑技能**：随安装附带（npm 软件包或 Moltbot.app）
2) **托管/本地技能**：`~/.clawdbot/skills`
3) **工作区技能**：`<workspace>/skills`

如果技能名称冲突，优先级为：

`<workspace>/skills`（最高）→ `~/.clawdbot/skills` → 捆绑技能（最低）

此外，您可以通过 `~/.clawdbot/moltbot.json` 中的 `skills.load.extraDirs` 配置额外的技能文件夹（最低优先级）。

## 每代理 vs 共享技能

在**多代理**设置中，每个代理都有自己的工作区。这意味着：

- **每代理技能**位于该代理的 `<workspace>/skills` 中。
- **共享技能**位于 `~/.clawdbot/skills`（托管/本地）中，对**同一机器上的所有代理**可见。
- **共享文件夹**也可以通过 `skills.load.extraDirs` 添加（最低优先级），如果您希望多个代理使用通用技能包。

如果同一技能名称存在于多个地方，则适用通常的优先级：工作区优先，然后是托管/本地，然后是捆绑。

## 插件 + 技能

插件可以通过在 `moltbot.plugin.json` 中列出 `skills` 目录（相对于插件根目录的路径）来附带自己的技能。插件技能在启用插件时加载，并参与正常的技能优先级规则。您可以通过插件配置条目上的 `metadata.moltbot.requires.config` 来限制它们。请参阅[插件](/plugin)了解发现/配置，以及[工具](/tools)了解这些技能教授的工具界面。

## ClawdHub（安装 + 同步）

ClawdHub 是 Moltbot 的公共技能注册表。在 https://clawdhub.com 浏览。使用它来发现、安装、更新和备份技能。
完整指南：[ClawdHub](/tools/clawdhub)。

常见流程：

- 将技能安装到您的工作区：
  - `clawdhub install <skill-slug>`
- 更新所有已安装的技能：
  - `clawdhub update --all`
- 同步（扫描 + 发布更新）：
  - `clawdhub sync --all`

默认情况下，`clawdhub` 安装到当前工作目录下的 `./skills` 中（或回退到配置的 Moltbot 工作区）。Moltbot 在下一次会话时将其视为 `<workspace>/skills`。

## 安全说明

- 将第三方技能视为**受信任的代码**。启用前请阅读它们。
- 对于不受信任的输入和风险工具，首选沙箱运行。请参阅[沙箱](/gateway/sandboxing)。
- `skills.entries.*.env` 和 `skills.entries.*.apiKey` 将密钥注入该代理运行的**主机**进程（而不是沙箱）。将密钥保留在提示和日志之外。
- 有关更广泛的威胁模型和检查清单，请参阅[安全性](/gateway/security)。

## 格式（AgentSkills + Pi 兼容）

`SKILL.md` 必须至少包含：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图像
---
```

说明：
- 我们遵循 AgentSkills 规范进行布局/意图。
- 嵌入式代理使用的解析器仅支持**单行** frontmatter 键。
- `metadata` 应该是**单行 JSON 对象**。
- 在说明中使用 `{baseDir}` 引用技能文件夹路径。
- 可选的 frontmatter 键：
  - `homepage` — 在 macOS 技能 UI 中显示为"Website"的 URL（也通过 `metadata.moltbot.homepage` 支持）。
  - `user-invocable` — `true|false`（默认：`true`）。当为 `true` 时，技能作为用户斜杠命令公开。
  - `disable-model-invocation` — `true|false`（默认：`false`）。当为 `true` 时，技能从模型提示中排除（仍然可以通过用户调用获得）。
  - `command-dispatch` — `tool`（可选）。设置为 `tool` 时，斜杠命令绕过模型并直接分派到工具。
  - `command-tool` — 当设置 `command-dispatch: tool` 时要调用的工具名称。
  - `command-arg-mode` — `raw`（默认）。对于工具分派，将原始参数字符串转发到工具（无核心解析）。

    工具使用参数调用：
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`。

## 限制（加载时过滤器）

Moltbot 在加载时使用 `metadata`（单行 JSON）**过滤技能**：

```markdown
---
name: nano-banana-pro
description: 通过 Gemini 3 Pro Image 生成或编辑图像
metadata: {"moltbot":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

`metadata.moltbot` 下的字段：
- `always: true` — 始终包含技能（跳过其他限制）。
- `emoji` — macOS 技能 UI 使用的可选表情符号。
- `homepage` — 在 macOS 技能 UI 中显示为"Website"的可选 URL。
- `os` — 平台的可选列表（`darwin`、`linux`、`win32`）。如果设置，技能仅在这些操作系统上符合条件。
- `requires.bins` — 列表；每个都必须存在于 `PATH` 上。
- `requires.anyBins` — 列表；至少一个必须存在于 `PATH` 上。
- `requires.env` — 列表；环境变量必须存在**或**在配置中提供。
- `requires.config` — 必须为真值的 `moltbot.json` 路径列表。
- `primaryEnv` — 与 `skills.entries.<name>.apiKey` 关联的环境变量名称。
- `install` — macOS 技能 UI 使用的安装程序规范可选数组（brew/node/go/uv/download）。

关于沙箱的说明：
- `requires.bins` 在技能加载时的**主机**上检查。
- 如果代理处于沙箱状态，二进制文件也必须存在于**容器内**。
  通过 `agents.defaults.sandbox.docker.setupCommand`（或自定义镜像）安装它。`setupCommand` 在容器创建后运行一次。
  包安装还需要网络出口、可写的根 FS 和沙箱中的 root 用户。
  示例：`summarize` 技能（`skills/summarize/SKILL.md`）需要在沙箱容器中使用 `summarize` CLI 才能在那里运行。

安装程序示例：

```markdown
---
name: gemini
description: 使用 Gemini CLI 进行编码帮助和 Google 搜索查找。
metadata: {"moltbot":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Install Gemini CLI (brew)"}]}}
---
```

说明：
- 如果列出了多个安装程序，网关会选择**单个**首选选项（brew 可用时为 brew，否则为 node）。
- 如果所有安装程序都是 `download`，Moltbot 会列出每个条目，以便您可以看到可用的工件。
- 安装程序规范可以包括 `os: ["darwin"|"linux"|"win32"]` 以按平台过滤选项。
- Node 安装遵守 `moltbot.json` 中的 `skills.install.nodeManager`（默认：npm；选项：npm/pnpm/yarn/bun）。
  这仅影响**技能安装**；网关运行时应该仍然是 Node（不建议将 Bun 用于 WhatsApp/Telegram）。
- Go 安装：如果缺少 `go` 并且可用 `brew`，网关首先通过 Homebrew 安装 Go，并在可能时将 `GOBIN` 设置为 Homebrew 的 `bin`。
- 下载安装：`url`（必需）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（默认：检测到 archive 时自动）、`stripComponents`、`targetDir`（默认：`~/.clawdbot/tools/<skillKey>`）。

如果不存在 `metadata.moltbot`，则技能始终符合条件（除非在配置中禁用或被捆绑技能的 `skills.allowBundled` 阻止）。

## 配置覆盖（`~/.clawdbot/moltbot.json`）

可以切换捆绑/托管技能并为其提供环境值：

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

注意：如果技能名称包含连字符，请引用键（JSON5 允许引用键）。

配置键默认匹配**技能名称**。如果技能定义 `metadata.moltbot.skillKey`，请在 `skills.entries` 下使用该键。

规则：
- `enabled: false` 禁用技能，即使它已捆绑/安装。
- `env`：仅在变量尚未在进程中设置时注入。
- `apiKey`：为声明 `metadata.moltbot.primaryEnv` 的技能提供的便利。
- `config`：可选的每技能自定义字段包；自定义键必须位于此处。
- `allowBundled`：**仅限捆绑**技能的可选允许列表。如果设置，仅列表中的捆绑技能符合条件（托管/工作区技能不受影响）。

## 环境注入（每代理运行）

当代理运行开始时，Moltbot：
1) 读取技能元数据。
2) 将任何 `skills.entries.<key>.env` 或 `skills.entries.<key>.apiKey` 应用于 `process.env`。
3) 使用**符合条件的**技能构建系统提示。
4) 在运行结束后恢复原始环境。

这是**限于代理运行的**，而不是全局 shell 环境。

## 会话快照（性能）

Moltbot 在**会话开始时**对符合条件的技能进行快照，并在同一会话的后续轮次中重用该列表。对技能或配置的更改在下一个新会话时生效。

当启用技能监视器或出现新的符合条件的远程节点时，技能也可以在会话中刷新（请参阅下文）。将其视为**热重载**：刷新的列表在下一个代理轮次时被拾取。

## 远程 macOS 节点（Linux 网关）

如果网关在 Linux 上运行，但连接了**macOS 节点**并且允许了 `system.run`（执行批准安全性未设置为 `deny`），当所需二进制文件存在于该节点上时，Moltbot 可以将仅限 macOS 的技能视为符合条件的。代理应该通过 `nodes` 工具（通常是 `nodes.run`）执行这些技能。

这依赖于节点报告其命令支持以及通过 `system.run` 进行 bin 探测。如果 macOS 节点稍后离线，技能仍然可见；调用可能会失败，直到节点重新连接。

## 技能监视器（自动刷新）

默认情况下，Moltbot 监视技能文件夹，并在 `SKILL.md` 文件更改时更新技能快照。在 `skills.load` 下配置：

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

## Token 影响（技能列表）

当技能符合条件时，Moltbot 将可用技能的紧凑 XML 列表注入到系统提示中（通过 `pi-coding-agent` 中的 `formatSkillsForPrompt`）。成本是确定性的：

- **基本开销（仅当 ≥1 技能时）**：195 个字符。
- **每技能**：97 个字符 + XML 转义的 `<name>`、`<description>` 和 `<location>` 值的长度。

公式（字符）：

```
总计 = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

说明：
- XML 转义将 `& < > " '` 扩展为实体（`&amp;`、`&lt;` 等），增加长度。
- Token 计数因模型分词器而异。粗略的 OpenAI 风格估计是约 4 个字符/token，因此 **97 个字符 ≈ 每技能 24 个 token** 加上您的实际字段长度。

## 托管技能生命周期

Moltbot 作为**捆绑技能**附带一组基线技能，作为安装（npm 软件包或 Moltbot.app）的一部分。`~/.clawdbot/skills` 的存在是为了本地覆盖（例如，固定/修补技能而不更改捆绑副本）。工作区技能是用户拥有的，并且在名称冲突时覆盖两者。

## 配置参考

请参阅[技能配置](/tools/skills-config)了解完整的配置架构。

## 寻找更多技能？

浏览 https://clawdhub.com。

---
