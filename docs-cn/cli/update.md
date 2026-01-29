---
summary: "`moltbot update` CLI 参考(安全更新源代码 + gateway 自动重启)"
read_when:
  - 您想安全地更新源代码检出
  - 您需要了解 `--update` 简写行为
---

# `moltbot update`

安全地更新 Moltbot 并在 stable/beta/dev 频道之间切换。

如果您通过 **npm/pnpm** 安装(全局安装,无 git 元数据),更新将通过 [Updating](/install/updating) 中的包管理器流程进行。

## 用法

```bash
moltbot update
moltbot update status
moltbot update wizard
moltbot update --channel beta
moltbot update --channel dev
moltbot update --tag beta
moltbot update --no-restart
moltbot update --json
moltbot --update
```

## 选项

- `--no-restart`:成功更新后跳过重启 Gateway 服务。
- `--channel <stable|beta|dev>`:设置更新频道(git + npm;持久化在配置中)。
- `--tag <dist-tag|version>`:仅为此更新覆盖 npm dist-tag 或版本。
- `--json`:打印机器可读的 `UpdateRunResult` JSON。
- `--timeout <seconds>`:每步超时(默认是 1200s)。

注意:降级需要确认,因为旧版本可能会破坏配置。

## `update status`

显示活动更新频道 + git 标签/分支/SHA(用于源代码检出),加上更新可用性。

```bash
moltbot update status
moltbot update status --json
moltbot update status --timeout 10
```

选项:
- `--json`:打印机器可读的状态 JSON。
- `--timeout <seconds>`:检查超时(默认是 3s)。

## `update wizard`

交互式流程,用于选择更新频道并确认在更新后是否重启 Gateway(默认是重启)。如果您在没有 git 检出的情况下选择 `dev`,它会提议创建一个。

## 它的作用

当您显式切换频道时(`--channel ...`),Moltbot 还会保持安装方法对齐:

- `dev` → 确保 git 检出(默认:`~/moltbot`,用 `CLAWDBOT_GIT_DIR` 覆盖),更新它,并从该检出安装全局 CLI。
- `stable`/`beta` → 使用匹配的 dist-tag 从 npm 安装。

## Git 检出流程

频道:

- `stable`:检出最新的非 beta 标签,然后构建 + 诊断。
- `beta`:检出最新的 `-beta` 标签,然后构建 + 诊断。
- `dev`:检出 `main`,然后获取 + rebase。

高级:

1. 需要干净的工作树(无未提交的更改)。
2. 切换到选定的频道(标签或分支)。
3. 获取上游(仅 dev)。
4. 仅 dev:在临时工作树中进行预检查 lint + TypeScript 构建;如果提示失败,则最多回退 10 个提交以查找最新的干净构建。
5. Rebase 到选定的提交(仅 dev)。
6. 安装依赖(首选 pnpm;npm 回退)。
7. 构建 + 构建 Control UI。
8. 运行 `moltbot doctor` 作为最终的"安全更新"检查。
9. 将插件同步到活动频道(dev 使用捆绑扩展;stable/beta 使用 npm)并更新 npm 安装的插件。

## `--update` 简写

`moltbot --update` 重写为 `moltbot update`(对 shell 和启动器脚本有用)。

## 另请参阅

- `moltbot doctor`(在 git 检出上提议先运行更新)
- [开发频道](/install/development-channels)
- [更新](/install/updating)
- [CLI 参考手册](/cli)
