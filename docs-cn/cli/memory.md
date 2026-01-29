---
summary: "`moltbot memory` CLI 参考(状态/索引/搜索)"
read_when:
  - 您想索引或搜索语义内存
  - 您正在调试内存可用性或索引
---

# `moltbot memory`

管理语义内存索引和搜索。
由活动的内存插件提供(默认:`memory-core`;设置 `plugins.slots.memory = "none"` 以禁用)。

相关:
- 内存概念: [Memory](/concepts/memory)
- 插件: [Plugins](/plugins)

## 示例

```bash
moltbot memory status
moltbot memory status --deep
moltbot memory status --deep --index
moltbot memory status --deep --index --verbose
moltbot memory index
moltbot memory index --verbose
moltbot memory search "release checklist"
moltbot memory status --agent main
moltbot memory index --agent main --verbose
```

## 选项

通用:

- `--agent <id>`:限定到单个 agent(默认:所有配置的 agent)。
- `--verbose`:在探测和索引期间发出详细日志。

注意事项:
- `memory status --deep` 探测向量 + 嵌入可用性。
- `memory status --deep --index` 在存储脏时运行重新索引。
- `memory index --verbose` 打印每个阶段的详细信息(提供商、模型、源、批处理活动)。
