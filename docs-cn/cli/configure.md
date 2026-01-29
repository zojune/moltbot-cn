---
summary: "`moltbot configure` CLI 参考(交互式配置提示)"
read_when:
  - 您想以交互方式调整凭据、设备或 agent 默认值
---

# `moltbot configure`

交互式提示以设置凭据、设备和 agent 默认值。

注意:**Model** 部分现在包括 `agents.defaults.models` 允许列表的多选(显示在 `/model` 和模型选择器中)。

提示:不带子命令的 `moltbot config` 打开相同的向导。使用 `moltbot config get|set|unset` 进行非交互式编辑。

相关:
- Gateway 配置参考: [Configuration](/gateway/configuration)
- 配置 CLI: [Config](/cli/config)

注意事项:
- 选择 Gateway 运行位置始终更新 `gateway.mode`。如果这就是您所需要的,您可以选择 "Continue" 而无需其他部分。
- 面向频道的服务(Slack/Discord/Matrix/Microsoft Teams)在设置期间提示频道/房间允许列表。您可以输入名称或 ID;向导尽可能将名称解析为 ID。

## 示例

```bash
moltbot configure
moltbot configure --section models --section channels
```
