---
summary: "完全卸载 Moltbot（CLI、服务、状态、工作区）"
read_when:
  - 您想从机器上删除 Moltbot
  - 卸载后网关服务仍在运行
---

# 卸载

两条路径：
- 如果 `moltbot` 仍安装，则为**简单路径**。
- 如果 CLI 已消失但服务仍在运行，则为**手动服务删除**。

## 简单路径（CLI 仍安装）

推荐：使用内置卸载程序：

```bash
moltbot uninstall
```

非交互式（自动化 / npx）：

```bash
moltbot uninstall --all --yes --non-interactive
npx -y moltbot uninstall --all --yes --non-interactive
```

手动步骤（结果相同）：

1) 停止网关服务：

```bash
moltbot gateway stop
```

2) 卸载网关服务（launchd/systemd/schtasks）：

```bash
moltbot gateway uninstall
```

3) 删除状态 + 配置：

```bash
rm -rf "${CLAWDBOT_STATE_DIR:-$HOME/.clawdbot}"
```

如果您将 `CLAWDBOT_CONFIG_PATH` 设置为状态目录之外的自定义位置，请也删除该文件。

4) 删除您的工作区（可选，删除代理文件）：

```bash
rm -rf ~/clawd
```

5) 删除 CLI 安装（选择您使用的一种）：

```bash
npm rm -g moltbot
pnpm remove -g moltbot
bun remove -g moltbot
```

6) 如果您安装了 macOS 应用：

```bash
rm -rf /Applications/Moltbot.app
```

注意：
- 如果您使用了配置文件（`--profile` / `CLAWDBOT_PROFILE`），请为每个状态目录重复步骤 3（默认为 `~/.clawdbot-<profile>`）。
- 在远程模式下，状态目录位于**网关主机**上，因此也在那里运行步骤 1-4。

## 手动服务删除（未安装 CLI）

如果网关服务保持运行但 `moltbot` 缺失，请使用此方法。

### macOS (launchd)

默认标签是 `bot.molt.gateway`（或 `bot.molt.<profile>`；旧的 `com.clawdbot.*` 可能仍然存在）：

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

如果您使用了配置文件，请将标签和 plist 名称替换为 `bot.molt.<profile>`。删除任何旧的 `com.clawdbot.*` plist（如果存在）。

### Linux (systemd 用户单元)

默认单元名称是 `moltbot-gateway.service`（或 `moltbot-gateway-<profile>.service`）：

```bash
systemctl --user disable --now moltbot-gateway.service
rm -f ~/.config/systemd/user/moltbot-gateway.service
systemctl --user daemon-reload
```

### Windows (计划任务)

默认任务名称是 `Moltbot Gateway`（或 `Moltbot Gateway (<profile>)`）。
任务脚本位于您的状态目录下。

```powershell
schtasks /Delete /F /TN "Moltbot Gateway"
Remove-Item -Force "$env:USERPROFILE\.clawdbot\gateway.cmd"
```

如果您使用了配置文件，请删除匹配的任务名称和 `~\.clawdbot-<profile>\gateway.cmd`。

## 正常安装 vs 源代码检出

### 正常安装（install.sh / npm / pnpm / bun）

如果您使用了 `https://molt.bot/install.sh` 或 `install.ps1`，CLI 是通过 `npm install -g moltbot@latest` 安装的。
使用 `npm rm -g moltbot` 删除它（或者如果您是那样安装的，使用 `pnpm remove -g` / `bun remove -g`）。

### 源代码检出（git clone）

如果您从仓库检出运行（`git clone` + `moltbot ...` / `bun run moltbot ...`）：

1) 在删除仓库之前**卸载网关服务**（使用上面的简单路径或手动服务删除）。
2) 删除仓库目录。
3) 如上所示删除状态 + 工作区。
