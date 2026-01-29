---
summary: "入站渠道位置解析（Telegram + WhatsApp）和上下文字段"
read_when:
  - 添加或修改渠道位置解析
  - 在 agent 提示或工具中使用位置上下文字段
---

# 渠道位置解析

Moltbot 将来自聊天渠道的共享位置规范化为：
- 附加到入站正文的人类可读文本，以及
- 自动回复上下文负载中的结构化字段。

目前支持：
- **Telegram**（位置标记 + 场所 + 实时位置）
- **WhatsApp**（locationMessage + liveLocationMessage）
- **Matrix**（带有 `geo_uri` 的 `m.location`）

## 文本格式
位置呈现为不带括号的友好行：

- 标记：
  - `📍 48.858844, 2.294351 ±12m`
- 命名地点：
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- 实时共享：
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

如果渠道包括标题/评论，它会附加在下一行：
```
📍 48.858844, 2.294351 ±12m
在这里见面
```

## 上下文字段
当存在位置时，这些字段被添加到 `ctx`：
- `LocationLat`（数字）
- `LocationLon`（数字）
- `LocationAccuracy`（数字，米；可选）
- `LocationName`（字符串；可选）
- `LocationAddress`（字符串；可选）
- `LocationSource`（`pin | place | live`）
- `LocationIsLive`（布尔值）

## 渠道说明
- **Telegram**：场所映射到 `LocationName/LocationAddress`；实时位置使用 `live_period`。
- **WhatsApp**：`locationMessage.comment` 和 `liveLocationMessage.caption` 被附加为标题行。
- **Matrix**：`geo_uri` 被解析为标记位置；忽略高度，`LocationIsLive` 始终为 false。
