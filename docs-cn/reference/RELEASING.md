---
summary: "npm + macOS 应用分步发布清单"
read_when:
  - 发布新的 npm 版本
  - 发布新的 macOS 应用版本
  - 在发布前验证元数据
---

# 发布清单（npm + macOS）

从仓库根目录使用 `pnpm` (Node 22+)。在标记/发布之前保持工作树整洁。

## 操作员触发
当操作员说"release"时，立即执行此预检（除非被阻止，否则没有额外问题）：
- 阅读本文档和 `docs/platforms/mac/release.md`。
- 从 `~/.profile` 加载环境并确认设置了 `SPARKLE_PRIVATE_KEY_FILE` + App Store Connect 变量（SPARKLE_PRIVATE_KEY_FILE 应位于 `~/.profile` 中）。
- 如果需要，请使用 `~/Library/CloudStorage/Dropbox/Backup/Sparkle` 中的 Sparkle 密钥。

1) **版本和元数据**
- [ ] 增加 `package.json` 版本（例如，`2026.1.27-beta.1`）。
- [ ] 运行 `pnpm plugins:sync` 以对齐扩展包版本 + 更改日志。
- [ ] 更新 CLI/版本字符串：[`src/cli/program.ts`](https://github.com/moltbot/moltbot/blob/main/src/cli/program.ts) 和 [`src/provider-web.ts`](https://github.com/moltbot/moltbot/blob/main/src/provider-web.ts) 中的 Baileys 用户代理。
- [ ] 确认包元数据（名称、描述、仓库、关键字、许可证）和 `bin` 映射指向 `moltbot` 的 [`moltbot.mjs`](https://github.com/moltbot/moltbot/blob/main/moltbot.mjs)。
- [ ] 如果依赖项发生更改，请运行 `pnpm install`，以便 `pnpm-lock.yaml` 是最新的。

2) **构建和工件**
- [ ] 如果 A2UI 输入发生更改，请运行 `pnpm canvas:a2ui:bundle` 并提交任何更新的 [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/moltbot/moltbot/blob/main/src/canvas-host/a2ui/a2ui.bundle.js)。
- [ ] `pnpm run build`（重新生成 `dist/`）。
- [ ] 验证 npm 包 `files` 包括所有必需的 `dist/*` 文件夹（特别是 `dist/node-host/**` 和 `dist/acp/**` 用于无头节点 + ACP CLI）。
- [ ] 确认 `dist/build-info.json` 存在并包含预期的 `commit` 哈希（CLI 横幅将其用于 npm 安装）。
- [ ] 可选：构建后 `npm pack --pack-destination /tmp`；检查 tarball 内容并将其保留用于 GitHub 发布（**不要**提交它）。

3) **更改日志和文档**
- [ ] 更新 `CHANGELOG.md` 中的面向用户的亮点（如果缺失则创建文件）；保持条目严格按版本降序排列。
- [ ] 确保 README 示例/标志与当前 CLI 行为匹配（特别是新命令或选项）。

4) **验证**
- [ ] `pnpm lint`
- [ ] `pnpm test`（如果需要覆盖输出，请运行 `pnpm test:coverage`）
- [ ] `pnpm run build`（测试后的最后健全性检查）
- [ ] `pnpm release:check`（验证 npm 打包内容）
- [ ] `CLAWDBOT_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke`（Docker 安装冒烟测试，快速路径；发布前必需）
  - 如果已知的上一个 npm 版本损坏，请设置 `CLAWDBOT_INSTALL_SMOKE_PREVIOUS=<last-good-version>` 或 `CLAWDBOT_INSTALL_SMOKE_SKIP_PREVIOUS=1` 用于预安装步骤。
- [ ] （可选）完整安装程序冒烟（增加非 root + CLI 覆盖）：`pnpm test:install:smoke`
- [ ] （可选）安装程序 E2E（Docker，运行 `curl -fsSL https://molt.bot/install.sh | bash`，onboards，然后运行真实工具调用）：
  - `pnpm test:install:e2e:openai`（需要 `OPENAI_API_KEY`）
  - `pnpm test:install:e2e:anthropic`（需要 `ANTHROPIC_API_KEY`）
  - `pnpm test:install:e2e`（需要两个密钥；运行两个提供商）
- [ ] （可选）如果您更改影响发送/接收路径，请抽查 web 网关。

5) **macOS 应用 (Sparkle)**
- [ ] 构建 + 签名 macOS 应用，然后将其压缩以进行分发。
- [ ] 生成 Sparkle appcast（通过 [`scripts/make_appcast.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/make_appcast.sh) 进行 HTML 笔记）并更新 `appcast.xml`。
- [ ] 准备应用程序 zip（和可选的 dSYM zip）以附加到 GitHub 发布。
- [ ] 遵循 [macOS 发布](/platforms/mac/release) 中的确切命令和所需的环境变量。
  - `APP_BUILD` 必须是数字 + 单调（无 `-beta`），以便 Sparkle 正确比较版本。
  - 如果要进行公证，请使用从 App Store Connect API 环境变量创建的 `moltbot-notary` 钥匙串配置文件（请参阅 [macOS 发布](/platforms/mac/release)）。

6) **发布 (npm)**
- [ ] 确认 git 状态干净；根据需要提交和推送。
- [ ] `npm login`（如果需要，请验证 2FA）。
- [ ] `npm publish --access public`（对于预发布版本使用 `--tag beta`）。
- [ ] 验证注册表：`npm view moltbot version`、`npm view moltbot dist-tags` 和 `npx -y moltbot@X.Y.Z --version`（或 `--help`）。

### 故障排除（来自 2.0.0-beta2 发布的说明）
- **npm pack/publish 挂起或产生巨大的 tarball**：`dist/Moltbot.app`（和发布 zip）中的 macOS 应用捆绑包被扫描到包中。通过 `package.json` `files` 白名单发布内容（包含 dist 子目录、docs、skills；排除应用捆绑包）。确认 `npm pack --dry-run` 未列出 `dist/Moltbot.app`。
- **dist-tags 的 npm auth web 循环**：使用传统身份验证以获取 OTP 提示：
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add moltbot@X.Y.Z latest`
- **`npx` 验证失败并显示 `ECOMPROMISED: Lock compromised`**：使用新鲜缓存重试：
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y moltbot@X.Y.Z --version`
- **标签在后期修复后需要重新指向**：强制更新并推送标签，然后确保 GitHub 发布资产仍然匹配：
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7) **GitHub 发布 + appcast**
- [ ] 标记并推送：`git tag vX.Y.Z && git push origin vX.Y.Z`（或 `git push --tags`）。
- [ ] 为 `vX.Y.Z` 创建/刷新 GitHub 发布，标题为 **`moltbot X.Y.Z`**（不仅仅是标签）；正文应包含该版本的**完整**更改日志部分（亮点 + 更改 + 修复），内联（无纯链接），并且**不得在正文中重复标题**。
- [ ] 附加工件：`npm pack` tarball（可选）、`Moltbot-X.Y.Z.zip` 和 `Moltbot-X.Y.Z.dSYM.zip`（如果生成）。
- [ ] 提交更新的 `appcast.xml` 并推送（Sparkle 从 main 提供内容）。
- [ ] 从干净的临时目录（无 `package.json`）运行 `npx -y moltbot@X.Y.Z send --help` 以确认安装/CLI 入口点正常工作。
- [ ] 公告/分享发布说明。

## 插件发布范围 (npm)

我们只在 `@moltbot/*` 范围下发布**现有的 npm 插件**。不在 npm 上的捆绑插件
保持**仅磁盘树**（仍发布在
`extensions/**` 中）。

派生列表的过程：
1. `npm search @moltbot --json` 并捕获包名称。
2. 与 `extensions/*/package.json` 名称进行比较。
3. 仅发布**交集**（已在 npm 上）。

当前 npm 插件列表（根据需要更新）：
- @moltbot/bluebubbles
- @moltbot/diagnostics-otel
- @moltbot/discord
- @moltbot/lobster
- @moltbot/matrix
- @moltbot/msteams
- @moltbot/nextcloud-talk
- @moltbot/nostr
- @moltbot/voice-call
- @moltbot/zalo
- @moltbot/zalouser

发布说明还必须指出**新的可选捆绑插件**，这些插件**默认未启用**（例如：`tlon`）。
