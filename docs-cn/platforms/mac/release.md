---
summary: "Moltbot macOS 发布清单（Sparkle feed、打包、签名）"
read_when:
  - 切换或验证 Moltbot macOS 发布
  - 更新 Sparkle appcast 或 feed 资产
---

# Moltbot macOS 发布（Sparkle）

此应用现在附带 Sparkle 自动更新。发布构建必须由 Developer ID 签名、压缩，并使用签名的 appcast 条目发布。

## 先决条件
- 已安装 Developer ID Application 证书（例如：`Developer ID Application: <Developer Name> (<TEAMID>)`）。
- Sparkle 私钥路径在环境中设置为 `SPARKLE_PRIVATE_KEY_FILE`（Sparkle ed25519 私钥的路径；公钥烘焙到 Info.plist 中）。如果缺失，请检查 `~/.profile`。
- Notary 凭据（钥匙串配置文件或 API 密钥）用于 `xcrun notarytool`，如果您想要 Gatekeeper 安全的 DMG/zip 分发。
  - 我们使用名为 `moltbot-notary` 的钥匙串配置文件，从 App Store Connect API 密钥环境变量在您的 shell 配置文件中创建：
    - `APP_STORE_CONNECT_API_KEY_P8`、`APP_STORE_CONNECT_KEY_ID`、`APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/moltbot-notary.p8`
    - `xcrun notarytool store-credentials "moltbot-notary" --key /tmp/moltbot-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- 已安装 `pnpm` 依赖项（`pnpm install --config.node-linker=hoisted`）。
- Sparkle 工具通过 SwiftPM 在 `apps/macos/.build/artifacts/sparkle/Sparkle/bin/`（`sign_update`、`generate_appcast` 等）中自动获取。

## 构建和打包
注意：
- `APP_BUILD` 映射到 `CFBundleVersion`/`sparkle:version`；保持数字 + 单调（无 `-beta`），否则 Sparkle 将其视为相等。
- 默认为当前架构（`$(uname -m)`）。对于发布/通用构建，设置 `BUILD_ARCHS="arm64 x86_64"`（或 `BUILD_ARCHS=all`）。
- 使用 `scripts/package-mac-dist.sh` 进行发布工件（zip + DMG + notarization）。使用 `scripts/package-mac-app.sh` 进行本地/开发打包。

```bash
# 从仓库根目录；设置发布 ID 以启用 Sparkle feed。
# APP_BUILD 必须是数字 + 单调的，以便进行 Sparkle 比较。
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip 以进行分发（包括 Sparkle delta 支持的资源分支）
ditto -c -k --sequesterRsrc --keepParent dist/Moltbot.app dist/Moltbot-2026.1.27-beta.1.zip

# 可选：还构建一个供人类使用的样式化 DMG（拖动到 /Applications）
scripts/create-dmg.sh dist/Moltbot.app dist/Moltbot-2026.1.27-beta.1.dmg

# 推荐：构建 + notarize/staple zip + DMG
# 首先，创建一次钥匙串配置文件：
#   xcrun notarytool store-credentials "moltbot-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=moltbot-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# 可选：将 dSYM 与发布一起发布
ditto -c -k --keepParent apps/macos/.build/release/Moltbot.app.dSYM dist/Moltbot-2026.1.27-beta.1.dSYM.zip
```

## Appcast 条目
使用发布笔记生成器，以便 Sparkle 呈现格式化的 HTML 笔记：
```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/Moltbot-2026.1.27-beta.1.zip https://raw.githubusercontent.com/moltbot/moltbot/main/appcast.xml
```
从 `CHANGELOG.md` 生成 HTML 发布笔记（通过 [`scripts/changelog-to-html.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/changelog-to-html.sh)）并将它们嵌入到 appcast 条目中。
在发布资产（zip + dSYM）旁边发布更新的 `appcast.xml` 时提交。

## 发布和验证
- 将 `Moltbot-2026.1.27-beta.1.zip`（和 `Moltbot-2026.1.27-beta.1.dSYM.zip`）上传到标签 `v2026.1.27-beta.1` 的 GitHub 发布。
- 确保原始 appcast URL 与烘焙的 feed 匹配：`https://raw.githubusercontent.com/moltbot/moltbot/main/appcast.xml`。
- 完整性检查：
  - `curl -I https://raw.githubusercontent.com/moltbot/moltbot/main/appcast.xml` 返回 200。
  - `curl -I <enclosure url>` 在资产上传后返回 200。
  - 在以前的公共构建上，从"关于"选项卡运行"检查更新..."并验证 Sparkle 正确安装新构建。

完成的定义：已签名的应用 + appcast 已发布，更新流程从已安装的旧版本工作，并且发布资产附加到 GitHub 发布。
