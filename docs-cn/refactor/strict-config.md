---
summary: "严格配置验证 + 仅 doctor 迁移"
read_when:
  - 设计或实现配置验证行为
  - 处理配置迁移或 doctor 工作流
  - 处理插件配置架构或插件加载限制
---
# 严格配置验证（仅 doctor 迁移）

## 目标
- **到处拒绝未知配置键**（根 + 嵌套）。
- **拒绝没有架构的插件配置**；不加载该插件。
- **删除加载时的旧版自动迁移**；迁移仅通过 doctor 运行。
- **在启动时自动运行 doctor（dry-run）**；如果无效，阻止非诊断命令。

## 非目标
- 加载时的向后兼容性（旧键不会自动迁移）。
- 未知键的静默丢弃。

## 严格验证规则
- 配置必须在每个级别完全匹配架构。
- 未知键是验证错误（根或嵌套处没有直通）。
- `plugins.entries.<id>.config` 必须由插件的架构验证。
  - 如果插件缺少架构，**拒绝插件加载**并显示明确的错误。
- 未知的 `channels.<id>` 键是错误，除非插件清单声明频道 id。
- 所有插件都需要插件清单（`moltbot.plugin.json`）。

## 插件架构强制执行
- 每个插件为其配置提供严格的 JSON Schema（在清单中内联）。
- 插件加载流程：
  1) 解析插件清单 + 架构（`moltbot.plugin.json`）。
  2) 根据架构验证配置。
  3) 如果缺少架构或配置无效：阻止插件加载，记录错误。
- 错误消息包括：
  - 插件 id
  - 原因（缺少架构 / 配置无效）
  - 验证失败的路径
- 禁用的插件保留其配置，但 Doctor + 日志会显示警告。

## Doctor 流程
- Doctor 在**每次**加载配置时运行（默认为 dry-run）。
- 如果配置无效：
  - 打印摘要 + 可操作的错误。
  - 指示：`moltbot doctor --fix`。
- `moltbot doctor --fix`：
  - 应用迁移。
  - 删除未知键。
  - 写入更新的配置。

## 命令限制（当配置无效时）
允许（仅诊断）：
- `moltbot doctor`
- `moltbot logs`
- `moltbot health`
- `moltbot help`
- `moltbot status`
- `moltbot gateway status`

其他所有内容都必须硬失败："配置无效。运行 `moltbot doctor --fix`。"

## 错误 UX 格式
- 单个摘要标题。
- 分组部分：
  - 未知键（完整路径）
  - 旧键 / 需要迁移
  - 插件加载失败（插件 id + 原因 + 路径）

## 实现接触点
- `src/config/zod-schema.ts`：删除根直通；到处严格对象。
- `src/config/zod-schema.providers.ts`：确保严格的频道架构。
- `src/config/validation.ts`：未知键失败；不应用旧版迁移。
- `src/config/io.ts`：删除旧版自动迁移；始终运行 doctor dry-run。
- `src/config/legacy*.ts`：将使用情况仅移动到 doctor。
- `src/plugins/*`：添加架构注册表 + 限制。
- `src/cli` 中的 CLI 命令限制。

## 测试
- 未知键拒绝（根 + 嵌套）。
- 插件缺少架构 → 插件加载被阻止，并显示明确的错误。
- 配置无效 → 网关启动被阻止，除了诊断命令。
- Doctor dry-run 自动；`doctor --fix` 写入更正的配置。
