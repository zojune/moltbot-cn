---
title: 形式化验证（安全模型）
summary: Moltbot 最高风险路径的机器检查安全模型。
permalink: /security/formal-verification/
---

# 形式化验证（安全模型）

此页面跟踪 Moltbot 的**形式化安全模型**（今天是 TLA+/TLC；根据需要更多）。

> 注意：一些旧链接可能指的是以前的项目名称。

**目标（北极星）**：提供机器检查的参数，证明 Moltbot 在明确假设下强制执行其预期的安全策略（授权、会话隔离、工具门控和错误配置安全）。

**这是什么（今天）**：一个可执行的、攻击者驱动的**安全回归套件**：
- 每个声明都有一个有限状态空间上的可运行模型检查。
- 许多声明都有一个配对的**负面模型**，它为现实的错误类别生成反例跟踪。

**这不是什么（还没有）**：证明"Moltbot 在所有方面都是安全的"或完整的 TypeScript 实现是正确的。

## 模型所在位置

模型维护在一个单独的仓库中：[vignesh07/clawdbot-formal-models](https://github.com/vignesh07/clawdbot-formal-models)。

## 重要警告

- 这些是**模型**，而不是完整的 TypeScript 实现。模型和代码之间的漂移是可能的。
- 结果受 TLC 探索的状态空间限制；"绿色"并不意味着超出建模假设和边界的安全性。
- 一些声明依赖于明确的环境假设（例如，正确的部署、正确的配置输入）。

## 重现结果

今天，通过在本地克隆模型仓库并运行 TLC 来重现结果（见下文）。未来的迭代可以提供：
- 具有公共工件的 CI 运行模型（反例跟踪、运行日志）
- 用于小型、有界检查的托管"运行此模型"工作流

入门：

```bash
git clone https://github.com/vignesh07/clawdbot-formal-models
cd clawdbot-formal-models

# 需要 Java 11+（TLC 在 JVM 上运行）。
# 仓库供应商一个固定的 `tla2tools.jar`（TLA+ 工具）并提供 `bin/tlc` + Make 目标。

make <target>
```

### Gateway 暴露和开放 gateway 错误配置

**声明：** 在没有认证的情况下绑定超过 loopback 可能导致远程妥协/增加暴露；令牌/密码阻止未经授权的攻击者（根据模型假设）。

- 绿色运行：
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 红色（预期）：
  - `make gateway-exposure-v2-negative`

另请参阅：模型仓库中的 `docs/gateway-exposure-matrix.md`。

### Nodes.run 管道（最高风险功能）

**声明：** `nodes.run` 需要（a）节点命令允许列表加上声明的命令和（b）实时批准（配置时）；批准被令牌化以防止重放（在模型中）。

- 绿色运行：
  - `make nodes-pipeline`
  - `make approvals-token`
- 红色（预期）：
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### 配对存储（DM 门控）

**声明：** 配对请求尊重 TTL 和待处理请求上限。

- 绿色运行：
  - `make pairing`
  - `make pairing-cap`
- 红色（预期）：
  - `make pairing-negative`
  - `make pairing-cap-negative`

### 入口门控（提及 + 控制命令绕过）

**声明：** 在需要提及的群组上下文中，未经授权的"控制命令"不能绕过提及门控。

- 绿色：
  - `make ingress-gating`
- 红色（预期）：
  - `make ingress-gating-negative`

### 路由/会话密钥隔离

**声明：** 来自不同对等方的 DM 不会合并到同一个会话中，除非显式链接/配置。

- 绿色：
  - `make routing-isolation`
- 红色（预期）：
  - `make routing-isolation-negative`


## v1++：额外的有界模型（并发、重试、跟踪正确性）

这些是后续模型，它们收紧了对真实世界失败模式（非原子更新、重试和消息扇出）的保真度。

### 配对存储并发 / 幂等性

**声明：** 配对存储应该强制执行 `MaxPending` 和幂等性，即使在交错下（即，"检查然后写入"必须是原子的/锁定的；刷新不应创建重复项）。

这意味着：
- 在并发请求下，您不能超过通道的 `MaxPending`。
- 对相同 `(channel, sender)` 的重复请求/刷新不应创建重复的实时待处理行。

- 绿色运行：
  - `make pairing-race`（原子/锁定上限检查）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 红色（预期）：
  - `make pairing-race-negative`（非原子 begin/commit 上限竞争）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### 入口跟踪相关性 / 幂等性

**声明：** 摄取应该在扇出之间保留跟踪相关性，并且在提供商重试下是幂等的。

这意味着：
- 当一个外部事件变成多个内部消息时，每个部分保持相同的跟踪/事件身份。
- 重试不会导致双重处理。
- 如果缺少提供商事件 ID，去重会回退到安全密钥（例如跟踪 ID）以避免丢失不同的事件。

- 绿色：
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 红色（预期）：
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### 路由 dmScope 优先级 + identityLinks

**声明：** 路由必须默认保持 DM 会话隔离，并且仅在显式配置时折叠会话（通道优先级 + 身份链接）。

这意味着：
- 通道特定的 dmScope 覆盖必须胜过全局默认值。
- identityLinks 应该仅在显式链接的组内折叠，而不是在不相关的对等方之间。

- 绿色：
  - `make routing-precedence`
  - `make routing-identitylinks`
- 红色（预期）：
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
