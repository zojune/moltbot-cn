---
summary: "安全地更新 Moltbot（全局安装或源代码），以及回滚策略"
read_when:
  - 更新 Moltbot
  - 更新后出现问题
---

# 更新

Moltbot 发展迅速（"1.0"之前）。请将更新视为基础设施部署：更新 → 运行检查 → 重启（或使用 `moltbot update`，它会重启）→ 验证。

## 推荐：重新运行网站安装程序（就地升级）

**首选**更新路径是从网站重新运行安装程序。它
检测现有安装、就地升级并在需要时运行 `moltbot doctor`。

```bash
curl -fsSL https://molt.bot/install.sh | bash
```

注意：
- 如果您不想让入门向导再次运行，请添加 `--no-onboard`。
- 对于**源代码安装**，请使用：
  ```bash
  curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  仅当仓库干净时，安装程序才会 `git pull --rebase`。
- 对于**全局安装**，脚本在底层使用 `npm install -g moltbot@latest`。
- 旧版说明：`moltbot` 仍然作为兼容性填充程序可用。

## 更新之前

- 了解您的安装方式：**全局**（npm/pnpm）vs **从源代码**（git clone）。
- 了解您的网关运行方式：**前台终端** vs **监督服务**（launchd/systemd）。
- 快照您的自定义配置：
  - 配置：`~/.clawdbot/moltbot.json`
  - 凭据：`~/.clawdbot/credentials/`
  - 工作区：`~/clawd`

## 更新（全局安装）

全局安装（选择一种）：

```bash
npm i -g moltbot@latest
```

```bash
pnpm add -g moltbot@latest
```

我们**不**推荐 Bun 用于网关运行时（WhatsApp/Telegram 存在 bug）。

要切换更新频道（git + npm 安装）：

```bash
moltbot update --channel beta
moltbot update --channel dev
moltbot update --channel stable
```

使用 `--tag <dist-tag|version>` 进行一次性安装标签/版本。

频道语义和发布说明参见 [开发频道](/install/development-channels)。

注意：在 npm 安装上，网关在启动时记录更新提示（检查当前频道标签）。通过 `update.checkOnStart: false` 禁用。

然后：

```bash
moltbot doctor
moltbot gateway restart
moltbot health
```

注意：
- 如果您的网关作为服务运行，首选 `moltbot gateway restart` 而不是杀死 PID。
- 如果您固定到特定版本，请参见下面的"回滚 / 固定"。

## 更新（`moltbot update`）

对于**源代码安装**（git 检出），首选：

```bash
moltbot update
```

它运行一个相对安全的更新流程：
- 需要干净的工作树。
- 切换到选定的频道（标签或分支）。
- 获取 + 基于配置的上游（开发频道）进行变基。
- 安装依赖、构建、构建控制 UI 并运行 `moltbot doctor`。
- 默认重启网关（使用 `--no-restart` 跳过）。

如果您通过 **npm/pnpm** 安装（无 git 元数据），`moltbot update` 将尝试通过您的软件包管理器更新。如果它无法检测安装，请改用"更新（全局安装）"。

## 更新（控制 UI / RPC）

控制 UI 具有**更新并重启**（RPC：`update.run`）。它：
1) 运行与 `moltbot update` 相同的源更新流程（仅 git 检出）。
2) 编写带有结构化报告的重启哨兵（stdout/stderr 尾部）。
3) 重启网关并向最后一个活动会话发送报告。

如果变基失败，网关将中止并重启而不应用更新。

## 更新（从源代码）

从仓库检出：

首选：

```bash
moltbot update
```

手动（大致等效）：

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # 首次运行时自动安装 UI 依赖
moltbot doctor
moltbot health
```

注意：
- 当您运行打包的 `moltbot` 二进制文件（[`moltbot.mjs`](https://github.com/moltbot/moltbot/blob/main/moltbot.mjs)）或使用 Node 运行 `dist/` 时，`pnpm build` 很重要。
- 如果您从仓库检出运行而没有全局安装，请对 CLI 命令使用 `pnpm moltbot ...`。
- 如果您直接从 TypeScript 运行（`pnpm moltbot ...`），通常不需要重新构建，但**配置迁移仍然适用** → 运行 doctor。
- 在全局和 git 安装之间切换很容易：安装另一种方式，然后运行 `moltbot doctor`，以便网关服务入口点被重写为当前安装。

## 始终运行：`moltbot doctor`

Doctor 是"安全更新"命令。它有意保持无聊：修复 + 迁移 + 警告。

注意：如果您在**源代码安装**（git 检出）上，`moltbot doctor` 将提议先运行 `moltbot update`。

它通常做的事情：
- 迁移已弃用的配置键 / 旧配置文件位置。
- 审核 DM 策略并在有风险的"开放"设置上发出警告。
- 检查网关健康状况并可以提议重启。
- 检测并将较旧的网关服务（launchd/systemd；旧的 schtasks）迁移到当前的 Moltbot 服务。
- 在 Linux 上，确保 systemd 用户 lingering（以便网关在注销后存活）。

详情：[Doctor](/gateway/doctor)

## 启动 / 停止 / 重启网关

CLI（无论操作系统如何都有效）：

```bash
moltbot gateway status
moltbot gateway stop
moltbot gateway restart
moltbot gateway --port 18789
moltbot logs --follow
```

如果您被监督：
- macOS launchd（应用捆绑的 LaunchAgent）：`launchctl kickstart -k gui/$UID/bot.molt.gateway`（使用 `bot.molt.<profile>`；旧的 `com.clawdbot.*` 仍然有效）
- Linux systemd 用户服务：`systemctl --user restart moltbot-gateway[-<profile>].service`
- Windows (WSL2)：`systemctl --user restart moltbot-gateway[-<profile>].service`
  - `launchctl`/`systemctl` 仅在安装服务时才有效；否则运行 `moltbot gateway install`。

运行手册 + 确切的服务标签：[网关运行手册](/gateway)

## 回滚 / 固定（当出现问题时）

### 固定（全局安装）

安装已知良好的版本（将 `<version>` 替换为最后一个工作的版本）：

```bash
npm i -g moltbot@<version>
```

```bash
pnpm add -g moltbot@<version>
```

提示：要查看当前发布的版本，请运行 `npm view moltbot version`。

然后重启 + 重新运行 doctor：

```bash
moltbot doctor
moltbot gateway restart
```

### 按日期固定（源代码）

从日期中选择一个提交（例如："截至 2026-01-01 的 main 状态"）：

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

然后重新安装依赖 + 重启：

```bash
pnpm install
pnpm build
moltbot gateway restart
```

如果您想稍后回到最新版本：

```bash
git checkout main
git pull
```

## 如果您被卡住

- 再次运行 `moltbot doctor` 并仔细阅读输出（它通常会告诉您修复方法）。
- 检查：[故障排除](/gateway/troubleshooting)
- 在 Discord 中询问：https://channels.discord.gg/clawd
