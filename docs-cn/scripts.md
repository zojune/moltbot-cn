---
summary: "仓库脚本：用途、范围和安全注意事项"
read_when:
  - 从仓库运行脚本
  - 在 ./scripts 下添加或更改脚本
---

# 脚本

`scripts/` 目录包含用于本地工作流程和运维任务的辅助脚本。
当任务显然与某个脚本相关时，请使用这些脚本；否则，首选 CLI。

## 约定

- 脚本是**可选的**，除非在文档或发布清单中引用
- 如果存在 CLI 界面，则首选 CLI（例如：认证监控使用 `moltbot models status --check`）
- 假设脚本是特定于主机的；在新机器上运行之前请先阅读它们

## Git 钩子

- `scripts/setup-git-hooks.js`：在 git 仓库内时为 `core.hooksPath` 提供尽力设置
- `scripts/format-staged.js`：暂存 `src/` 和 `test/` 文件的预提交格式化程序

## 认证监控脚本

认证监控脚本在此处记录：
[/automation/auth-monitoring](/automation/auth-monitoring)

## 添加脚本时

- 保持脚本的专注性和文档化
- 在相关文档中添加简短条目（如果缺失则创建一个）
