---
summary: "菜单栏状态逻辑和向用户展示的内容"
read_when:
  - 调整 mac 菜单 UI 或状态逻辑
---
# 菜单栏状态逻辑

## 显示的内容
- 我们在菜单栏图标和菜单的第一个状态行中显示当前代理工作状态。
- 工作处于活动状态时隐藏健康状态；所有会话空闲时返回。
- 菜单中的"节点"块仅列出**设备**（通过 `node.list` 配对的节点），而不是客户端/状态条目。
- 当提供程序使用情况快照可用时，上下文下的"使用情况"部分会出现。

## 状态模型
- 会话：事件带有 `runId`（每次运行）加上有效负载中的 `sessionKey`。"main"会话是键 `main`；如果缺失，我们将回退到最近更新的会话。
- 优先级：main 总是获胜。如果 main 处于活动状态，则会立即显示其状态。如果 main 处于空闲状态，则显示最近活动的非 main 会话。我们不会在活动中途翻转；只有在当前会话进入空闲或 main 变为活动状态时才会切换。
- 活动类型：
  - `job`：高级命令执行（`state: started|streaming|done|error`）。
  - `tool`：带有 `toolName` 和可选 `meta`/`args` 的 `phase: start|result`。

## IconState 枚举 (Swift)
- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)`（调试覆盖）

### ActivityKind → 字形
- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- 默认 → 🛠️

### 视觉映射
- `idle`：正常生物。
- `workingMain`：带有字形的徽章，完全着色，腿部"工作"动画。
- `workingOther`：带有字形的徽章，静音着色，没有急跑。
- `overridden`：无论活动如何，都使用所选的字形/着色。

## 状态行文本（菜单）
- 当工作处于活动状态时：`<Session role> · <activity label>`
  - 示例：`Main · exec: pnpm test`、`Other · read: apps/macos/Sources/Moltbot/AppState.swift`。
- 空闲时：回退到健康摘要。

## 事件摄取
- 来源：控制通道 `agent` 事件（`ControlChannel.handleAgentEvent`）。
- 解析字段：
  - `stream: "job"` 带有用于开始/停止的 `data.state`。
  - `stream: "tool"` 带有 `data.phase`、`name`、可选的 `meta`/`args`。
- 标签：
  - `exec`：`args.command` 的第一行。
  - `read`/`write`：缩短的路径。
  - `edit`：路径加上从 `meta`/diff 计数推断的更改类型。
  - 回退：工具名称。

## 调试覆盖
- 设置 ▸ 调试 ▸ "图标覆盖"选择器：
  - `System (auto)`（默认）
  - `Working: main`（按工具类型）
  - `Working: other`（按工具类型）
  - `Idle`
- 通过 `@AppStorage("iconOverride")` 存储；映射到 `IconState.overridden`。

## 测试清单
- 触发主会话作业：验证图标立即切换并显示主标签。
- 在 main 空闲时触发非主会话作业：图标/状态显示非主；在完成之前保持稳定。
- 在其他活动时启动 main：图标立即翻转到 main。
- 快速工具爆发：确保徽章不会闪烁（工具结果的 TTL 宽限）。
- 所有会话空闲后重新出现健康行。
