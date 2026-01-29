---
summary: "稳定版、测试版和开发频道：语义说明、切换和标签"
read_when:
  - 您想要在稳定版/测试版/开发版之间切换
  - 您正在标记或发布预发布版本
---

# 开发频道

最后更新：2026-01-21

Moltbot 提供三个更新频道：

- **stable（稳定版）**: npm dist-tag `latest`。
- **beta（测试版）**: npm dist-tag `beta`（正在测试的构建）。
- **dev（开发版）**: `main` 的移动头部（git）。npm dist-tag：`dev`（发布时）。

我们将构建发布到 **beta**，进行测试，然后**将经过验证的构建提升到 `latest`**
而不更改版本号 — dist-tags 是 npm 安装的权威来源。

## 切换频道

Git 检出：

```bash
moltbot update --channel stable
moltbot update --channel beta
moltbot update --channel dev
```

- `stable`/`beta` 检出最新的匹配标签（通常是同一个标签）。
- `dev` 切换到 `main` 并基于上游进行变基。

npm/pnpm 全局安装：

```bash
moltbot update --channel stable
moltbot update --channel beta
moltbot update --channel dev
```

这通过相应的 npm dist-tag（`latest`、`beta`、`dev`）进行更新。

当您使用 `--channel` **显式**切换频道时，Moltbot 还会
对齐安装方法：

- `dev` 确保 git 检出（默认 `~/moltbot`，可通过 `CLAWDBOT_GIT_DIR` 覆盖），
  更新它，并从该检出安装全局 CLI。
- `stable`/`beta` 使用匹配的 dist-tag 从 npm 安装。

提示：如果您想要稳定版 + 开发版并行，请保留两个克隆并将您的网关指向稳定版。

## 插件和频道

当您使用 `moltbot update` 切换频道时，Moltbot 还会同步插件来源：

- `dev` 优先使用来自 git 检出的捆绑插件。
- `stable` 和 `beta` 恢复 npm 安装的插件包。

## 标记最佳实践

- 标记您希望 git 检出落地的版本（`vYYYY.M.D` 或 `vYYYY.M.D-<patch>`）。
- 保持标签不可变：永远不要移动或重用标签。
- npm dist-tags 仍然是 npm 安装的权威来源：
  - `latest` → 稳定版
  - `beta` → 候选构建
  - `dev` → main 快照（可选）

## macOS 应用可用性

测试版和开发版构建**可能不**包含 macOS 应用版本。没关系：

- git 标签和 npm dist-tag 仍然可以发布。
- 在发布说明或更新日志中说明"此测试版没有 macOS 构建"。
