---
summary: "Moltbot presence 条目如何生成、合并和显示"
read_when:
  - 调试 Instances 选项卡
  - 调查重复或过时的实例行
  - 更改 gateway WS 连接或系统事件信标
---
# Presence

Moltbot "presence"是一个轻量级、尽力而为的视图，包括：
- **Gateway** 本身，以及
- **连接到 Gateway 的客户端**（mac 应用、WebChat、CLI 等）

Presence 主要用于渲染 macOS 应用的**Instances**选项卡并提供快速的操作员可见性。

## Presence 字段（显示内容）

Presence 条目是具有以下字段的结构化对象：

- `instanceId`（可选但强烈推荐）：稳定的客户端身份（通常是 `connect.client.instanceId`）
- `host`：人性化的主机名
- `ip`：尽力而为的 IP 地址
- `version`：客户端版本字符串
- `deviceFamily` / `modelIdentifier`：硬件提示
- `mode`：`ui`、`webchat`、`cli`、`backend`、`probe`、`test`、`node` 等
- `lastInputSeconds`："自上次用户输入以来的秒数"（如果已知）
- `reason`：`self`、`connect`、`node-connected`、`periodic` 等
- `ts`：最后更新时间戳（自纪元以来的毫秒数）

## 生产者（presence 的来源）

Presence 条目由多个来源生成并**合并**。

### 1) Gateway 自身条目

Gateway 在启动时总是播种一个"self"条目，以便 UI 在任何客户端连接之前显示 gateway 主机。

### 2) WebSocket 连接

每个 WS 客户端以 `connect` 请求开始。成功握手后，Gateway 为该连接插入一个 presence 条目。

#### 为什么一次性 CLI 命令不显示

CLI 通常连接短暂的、一次性的命令。为了避免垃圾邮件发送 Instances 列表，`client.mode === "cli"` **不会**转换为 presence 条目。

### 3) `system-event` 信标

客户端可以通过 `system-event` 方法发送更丰富的定期信标。mac 应用使用此功能报告主机名、IP 和 `lastInputSeconds`。

### 4) 节点连接（role: node）

当节点通过 Gateway WebSocket 以 `role: node` 连接时，Gateway 为该节点插入一个 presence 条目（与其他 WS 客户端相同的流程）。

## 合并 + 去重规则（为什么 `instanceId` 很重要）

Presence 条目存储在单个内存映射中：

- 条目由 **presence key** 键控。
- 最好的键是稳定的 `instanceId`（来自 `connect.client.instanceId`），它在重启后仍然存在。
- 键不区分大小写。

如果客户端在没有稳定的 `instanceId` 的情况下重新连接，它可能会显示为**重复**行。

## TTL 和有界大小

Presence 是有意短暂的：

- **TTL：**超过 5 分钟的条目被修剪
- **最大条目数：**200（最旧的先删除）

这使列表保持新鲜并避免无界内存增长。

## 远程/隧道警告（环回 IP）

当客户端通过 SSH 隧道/本地端口转发连接时，Gateway 可能会将远程地址视为 `127.0.0.1`。为了避免覆盖良好的客户端报告的 IP，环回远程地址将被忽略。

## 消费者

### macOS Instances 选项卡

macOS 应用渲染 `system-presence` 的输出，并根据最后更新的年龄应用小型状态指示器（Active/Idle/Stale）。

## 调试提示

- 要查看原始列表，请对 Gateway 调用 `system-presence`。
- 如果看到重复项：
  - 确认客户端在握手中发送稳定的 `client.instanceId`
  - 确认定期信标使用相同的 `instanceId`
  - 检查连接派生的条目是否缺少 `instanceId`（预期会有重复）
