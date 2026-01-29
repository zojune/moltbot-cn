---
summary: "插件清单 + JSON 模式要求（严格配置验证）"
read_when:
  - 您正在构建 Moltbot 插件
  - 您需要提供插件配置模式或调试插件验证错误
---
# 插件清单 (moltbot.plugin.json)

每个插件**必须**在**插件根目录**中提供一个 `moltbot.plugin.json` 文件。
Moltbot 使用此清单来验证配置，**而无需执行插件代码**。缺少或无效的清单被视为插件错误，并会阻止配置验证。

查看完整的插件系统指南：[Plugins](/plugin)

## 必需字段

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

必需键：
- `id`（字符串）：规范插件 id。
- `configSchema`（对象）：插件配置的 JSON 模式（内联）。

可选键：
- `kind`（字符串）：插件种类（例如：`"memory"`）。
- `channels`（数组）：由此插件注册的频道 id（例如：`["matrix"]`）。
- `providers`（数组）：由此插件注册的提供商 id。
- `skills`（数组）：要加载的技能目录（相对于插件根目录）。
- `name`（字符串）：插件的显示名称。
- `description`（字符串）：简短的插件摘要。
- `uiHints`（对象）：用于 UI 呈现的配置字段标签/占位符/敏感标志。
- `version`（字符串）：插件版本（信息性）。

## JSON 模式要求

- **每个插件必须提供一个 JSON 模式**，即使它不接受任何配置。
- 空模式是可以接受的（例如：`{ "type": "object", "additionalProperties": false }`）。
- 模式在配置读/写时验证，而不是在运行时。

## 验证行为

- 未知的 `channels.*` 键是**错误**，除非频道 id 由插件清单声明。
- `plugins.entries.<id>`、`plugins.allow`、`plugins.deny` 和 `plugins.slots.*` 必须引用**可发现的**插件 id。未知的 id 是**错误**。
- 如果插件已安装但清单或模式损坏或缺失，验证将失败，Doctor 会报告插件错误。
- 如果插件配置存在但插件被**禁用**，配置将被保留，并且 Doctor + 日志中会显示**警告**。

## 注意事项

- 清单**对所有插件都是必需的**，包括本地文件系统加载。
- 运行时仍然单独加载插件模块；清单仅用于发现 + 验证。
- 如果您的插件依赖于原生模块，请记录构建步骤和任何包管理器允许列表要求（例如：pnpm `allow-build-scripts` + `pnpm rebuild <package>`）。
