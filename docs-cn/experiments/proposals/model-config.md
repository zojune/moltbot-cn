---
summary: "Exploration: 模型 配置, 认证 配置文件, and fallback 行为"
read_when: 
  - Exploring future model selection + auth profile ideas
---
# 模型 配置 (Exploration)

This document captures **ideas** for future 模型 配置. It is not a
shipping spec. For current 行为, 参见:
- [模型](/concepts/模型)
- [模型 failover](/concepts/模型-failover)
- [OAuth + 配置文件](/concepts/oauth)

## Motivation

Operators want:
- Multiple 认证 配置文件 per 提供商 (personal vs work).
- Simple `/model` selection with predictable fallbacks.
- Clear separation between text 模型 and image-capable 模型.

## Possible direction (high level)

- Keep model selection simple: `provider/model` with 可选 aliases.
- Let 提供商 have multiple 认证 配置文件, with an explicit order.
- Use a global fallback list so all 会话 fail over consistently.
- Only override image routing when explicitly configured.

## Open questions

- Should 配置文件 rotation be per-提供商 or per-模型?
- How should the UI surface 配置文件 selection for a 会话?
- What is the safest 迁移 路径 from legacy 配置 keys?
