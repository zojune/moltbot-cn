---
summary: "为在 Moltbot macOS 应用上工作的开发人员设置指南"
read_when:
  - 设置 macOS 开发环境
---
# macOS 开发人员设置

本指南涵盖了从源代码构建和运行 Moltbot macOS 应用所需的必要步骤。

## 前提条件

在构建应用之前，确保您已安装以下内容：

1.  **Xcode 26.2+**：Swift 开发所需。
2.  **Node.js 22+ & pnpm**：网关、CLI 和打包脚本所需。

## 1. 安装依赖项

安装项目范围的依赖项：

```bash
pnpm install
```

## 2. 构建和打包应用

要构建 macOS 应用并将其打包到 `dist/Moltbot.app` 中，请运行：

```bash
./scripts/package-mac-app.sh
```

如果您没有 Apple Developer ID 证书，脚本将自动使用 **临时签名**（`-`）。

有关开发运行模式、签名标志和 Team ID 故障排除，请参阅 macOS 应用 README：
https://github.com/moltbot/moltbot/blob/main/apps/macos/README.md

> **注意**：临时签名的应用可能会触发安全提示。如果应用立即崩溃并显示"Abort trap 6"，请参阅[故障排除](#troubleshooting)部分。

## 3. 安装 CLI

macOS 应用需要全局 `moltbot` CLI 安装来管理后台任务。

**要安装它（推荐）：**
1.  打开 Moltbot 应用。
2.  转到**常规**设置选项卡。
3.  点击**"安装 CLI"**。

或者，手动安装：
```bash
npm install -g moltbot@<version>
```

## 故障排除

### 构建失败：工具链或 SDK 不匹配
macOS 应用构建期望最新的 macOS SDK 和 Swift 6.2 工具链。

**系统依赖项（必需）：**
- **软件更新中可用的最新 macOS 版本**（Xcode 26.2 SDK 所需）
- **Xcode 26.2**（Swift 6.2 工具链）

**检查：**
```bash
xcodebuild -version
xcrun swift --version
```

如果版本不匹配，请更新 macOS/Xcode 并重新运行构建。

### 应用在授予权限时崩溃
如果您在尝试允许**语音识别**或**麦克风**访问时应用崩溃，可能是由于 TCC 缓存损坏或签名不匹配。

**修复：**
1. 重置 TCC 权限：
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. 如果失败，请暂时更改 [`scripts/package-mac-app.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/package-mac-app.sh) 中的 `BUNDLE_ID`，以强制从 macOS 获得"干净的一页"。

### 网关"正在启动..." indefinitely
如果网关状态保持"正在启动..."，请检查僵尸进程是否持有关口：

```bash
moltbot gateway status
moltbot gateway stop

# 如果您不使用 LaunchAgent（开发模式/手动运行），查找监听器：
lsof -nP -iTCP:18789 -sTCP:LISTEN
```
如果手动运行持有关口，请停止该进程（Ctrl+C）。作为最后手段，杀死您在上面找到的 PID。
