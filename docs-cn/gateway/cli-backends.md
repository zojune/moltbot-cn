---
summary: "CLI 后端:通过本地 AI CLI 的纯文本后备"
read_when:
  - 当 API 提供商失败时需要可靠的后备
  - 您正在运行 Claude Code CLI 或其他本地 AI CLI 并希望重用它们
  - 需要纯文本、无工具的路径,但仍支持会话和图像
---
# CLI 后端(后备运行时)

Moltbot 可以运行**本地 AI CLI**作为**纯文本后备**,当 API 提供商宕机、
速率限制或暂时异常时。这是有意的保守做法:

- **工具被禁用**(无工具调用)。
- **文本入 → 文本出**(可靠)。
- **支持会话**(因此后续轮次保持连贯)。
- **如果 CLI 接受图像路径,可以传递图像**。

这被设计为**安全网**而不是主要路径。当您
想要"始终有效"的文本响应而不依赖外部 API 时使用它。

## 对初学者友好的快速入门

您可以在**无需任何配置**的情况下使用 Claude Code CLI(Moltbot 附带内置默认值):

```bash
moltbot agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI 也可以开箱即用:

```bash
moltbot agent --message "hi" --model codex-cli/gpt-5.2-codex
```

如果您的 gateway 在 launchd/systemd 下运行并且 PATH 最小,只需添加
命令路径:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

就是这样。除了 CLI 本身外,不需要密钥或额外的认证配置。

## 将其用作后备

将 CLI 后端添加到您的后备列表中,以便它仅在主要模型失败时运行:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

注意事项:
- 如果您使用 `agents.defaults.models`(允许列表),必须包括 `claude-cli/...`。
- 如果主要提供商失败(认证、速率限制、超时),Moltbot 将
  尝试 CLI 后端。

## 配置概述

所有 CLI 后端位于:

```
agents.defaults.cliBackends
```

每个条目由**提供商 id** 键控(例如 `claude-cli`、`my-cli`)。
提供商 id 成为模型引用的左侧:

```
<provider>/<model>
```

### 示例配置

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```

## 工作原理

1) 根据提供商前缀(`claude-cli/...`)**选择后端**。
2) 使用相同的 Moltbot 提示 + 工作区上下文**构建系统提示**。
3) **执行 CLI**,带有会话 id(如果支持),以便历史记录保持一致。
4) **解析输出**(JSON 或纯文本)并返回最终文本。
5) **持久化每个后端的会话 id**,以便后续轮次重用相同的 CLI 会话。

## 会话

- 如果 CLI 支持会话,请设置 `sessionArg`(例如 `--session-id`)或
  `sessionArgs`(占位符 `{sessionId}`),当 ID 需要插入
  到多个标志中时。
- 如果 CLI 使用带有不同标志的**恢复子命令**,请设置
  `resumeArgs`(在恢复时替换 `args`)和可选的 `resumeOutput`
  (用于非 JSON 恢复)。
- `sessionMode`:
  - `always`:始终发送会话 id(如果没有存储则为新的 UUID)。
  - `existing`:仅在之前存储了会话 id 时才发送会话 id。
  - `none`:从不发送会话 id。

## 图像(透传)

如果您的 CLI 接受图像路径,请设置 `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

Moltbot 将 base64 图像写入临时文件。如果设置了 `imageArg`,这些
路径将作为 CLI 参数传递。如果缺少 `imageArg`,Moltbot 会将
文件路径附加到提示(路径注入),这对于从纯路径自动
加载本地文件的 CLI 来说已经足够了(Claude Code CLI 行为)。

## 输入/输出

- `output: "json"`(默认)尝试解析 JSON 并提取文本 + 会话 id。
- `output: "jsonl"` 解析 JSONL 流(Codex CLI `--json`)并提取
  最后的代理消息以及 `thread_id`(如果存在)。
- `output: "text"` 将 stdout 视为最终响应。

输入模式:
- `input: "arg"`(默认)将提示作为最后一个 CLI 参数传递。
- `input: "stdin"` 通过 stdin 发送提示。
- 如果提示很长并且设置了 `maxPromptArgChars`,则使用 stdin。

## 默认值(内置)

Moltbot 为 `claude-cli` 附带默认值:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

Moltbot 还为 `codex-cli` 附带默认值:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

仅在需要时覆盖(常见:绝对 `command` 路径)。

## 限制

- **无 Moltbot 工具**(CLI 后端从不接收工具调用)。某些 CLI
  可能仍运行自己的代理工具。
- **无流式传输**(收集 CLI 输出然后返回)。
- **结构化输出**取决于 CLI 的 JSON 格式。
- **Codex CLI 会话**通过文本输出恢复(无 JSONL),这比
  初始 `--json` 运行的结构化程度低。Moltbot 会话仍正常
  工作。

## 故障排除

- **未找到 CLI**:将 `command` 设置为完整路径。
- **模型名称错误**:使用 `modelAliases` 将 `provider/model` → CLI 模型。
- **无会话连续性**:确保设置了 `sessionArg` 并且 `sessionMode` 不为
  `none`(Codex CLI 目前无法通过 JSON 输出恢复)。
- **图像被忽略**:设置 `imageArg`(并验证 CLI 支持文件路径)。
