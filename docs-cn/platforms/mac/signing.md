---
summary: "打包脚本生成的 macOS 调试构建的签名步骤"
read_when:
  - 构建或签名 mac 调试构建
---
# mac 签名（调试构建）

此应用通常从 [`scripts/package-mac-app.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/package-mac-app.sh) 构建，它现在：
- 设置稳定的调试捆绑包标识符：`bot.molt.mac.debug`
- 使用该捆绑包 id 写入 Info.plist（通过 `BUNDLE_ID=...` 覆盖）
- 调用 [`scripts/codesign-mac-app.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/codesign-mac-app.sh) 来签名主二进制文件和应用捆绑包，以便 macOS 将每次重建视为相同的签名捆绑包并在重建之间保留 TCC 权限（通知、辅助功能、屏幕录制、麦克风、语音）。对于稳定的权限，请使用真实的签名身份；临时签名是选择加入的且脆弱（请参阅 [macOS 权限](/platforms/mac/permissions)）。
- 默认使用 `CODESIGN_TIMESTAMP=auto`；它为 Developer ID 签名启用受信任的时间戳。设置 `CODESIGN_TIMESTAMP=off` 以跳过时间戳（离线调试构建）。
- 将构建元数据注入 Info.plist：`MoltbotBuildTimestamp` (UTC) 和 `MoltbotGitCommit`（短哈希），以便关于窗格可以显示构建、git 和调试/发布频道。
- **打包需要 Node 22+**：脚本运行 TS 构建和控制 UI 构建。
- 从环境中读取 `SIGN_IDENTITY`。将 `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（或您的 Developer ID Application 证书）添加到您的 shell rc，以便始终使用您的证书签名。临时签名需要通过 `ALLOW_ADHOC_SIGNING=1` 或 `SIGN_IDENTITY="-"` 显式选择加入（不建议用于权限测试）。
- 在签名后运行 Team ID 审计，如果应用捆绑包内的任何 Mach-O 由不同的 Team ID 签名，则会失败。设置 `SKIP_TEAM_ID_CHECK=1` 以绕过。

## 用法

```bash
# 从仓库根目录
scripts/package-mac-app.sh               # 自动选择身份；如果未找到则错误
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # 真实证书
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # 临时（权限不会保留）
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # 显式临时（相同警告）
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 仅开发 Sparkle Team ID 不匹配的变通方法
```

### 临时签名说明
使用 `SIGN_IDENTITY="-"`（临时）签名时，脚本会自动禁用**Hardened Runtime**（`--options runtime`）。这是防止应用在尝试加载不共享相同 TeamID 的嵌入式框架（如 Sparkle）时崩溃所必需的。临时签名还会破坏 TCC 权限持久性；有关恢复步骤，请参阅 [macOS 权限](/platforms/mac/permissions)。

## 关于的构建元数据

`package-mac-app.sh` 使用以下内容标记捆绑包：
- `MoltbotBuildTimestamp`：打包时的 ISO8601 UTC
- `MoltbotGitCommit`：短 git 哈希（或 `unknown`（如果不可用））

关于选项卡读取这些键以显示版本、构建日期、git 提交以及它是否是调试构建（通过 `#if DEBUG`）。在代码更改后运行打包程序以刷新这些值。

## 原因

TCC 权限与捆绑包标识符*和*代码签名相关联。具有更改 UUID 的未签名调试构建导致 macOS 在每次重建后忘记授予。签名二进制文件（默认临时）并保持固定的捆绑包 id/路径（`dist/Moltbot.app`）在构建之间保留授予，匹配 VibeTunnel 方法。
