---
summary: "使用跟踪表面和凭据要求"
read_when:
  - 您正在连接提供程序使用/配额表面
  - 您需要解释使用跟踪行为或认证要求
---
# 使用跟踪

## 它是什么
- 直接从其使用端点提取提供程序使用/配额。
- 没有估算成本；仅提供程序报告的窗口。

## 它显示在哪里
- 聊天中的 `/status`：带有会话令牌 + 估算成本的表情符号丰富状态卡（仅 API 密钥）。当可用时，提供程序使用显示**当前模型提供程序**。
- 聊天中的 `/usage off|tokens|full`：每次回复使用页脚（OAuth 仅显示令牌）。
- 聊天中的 `/usage cost`：从 Moltbot 会话日志聚合的本地成本摘要。
- CLI：`moltbot status --usage` 打印完整的每个提供程序细分。
- CLI：`moltbot channels list` 在提供程序配置旁边打印相同的使用快照（使用 `--no-usage` 跳过）。
- macOS 菜单栏：Context 下的"Usage"部分（仅可用时）。

## 提供程序 + 凭据
- **Anthropic (Claude)**：认证配置文件中的 OAuth 令牌。
- **GitHub Copilot**：认证配置文件中的 OAuth 令牌。
- **Gemini CLI**：认证配置文件中的 OAuth 令牌。
- **Antigravity**：认证配置文件中的 OAuth 令牌。
- **OpenAI Codex**：认证配置文件中的 OAuth 令牌（存在时使用 accountId）。
- **MiniMax**：API 密钥（编码计划密钥；`MINIMAX_CODE_PLAN_KEY` 或 `MINIMAX_API_KEY`）；使用 5 小时编码计划窗口。
- **z.ai**：通过 env/config/auth 存储的 API 密钥。

如果不存在匹配的 OAuth/API 凭据，则隐藏使用情况。
