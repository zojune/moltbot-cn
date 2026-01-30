# 仓库指南
- 仓库：https://github.com/moltbot/moltbot
- GitHub issues/评论/PR 评论：使用字面量多行字符串或 `-F - <<'EOF'`（或 $'...'）来表示真正的换行；永远不要嵌入 "\\n"。

## 项目结构与模块组织
- 源代码：`src/`（CLI 连接在 `src/cli`，命令在 `src/commands`，web 提供者在 `src/provider-web.ts`，基础设施在 `src/infra`，媒体管道在 `src/media`）。
- 测试：与源文件并列放置的 `*.test.ts`。
- 文档：`docs/`（图片、队列、Pi 配置）。构建输出位于 `dist/`。
- 插件/扩展：位于 `extensions/*`（工作区包）。保持仅插件的依赖在扩展的 `package.json` 中；除非核心使用它们，否则不要将它们添加到根 `package.json`。
- 插件：安装时在插件目录运行 `npm install --omit=dev`；运行时依赖必须位于 `dependencies` 中。避免在 `dependencies` 中使用 `workspace:*`（npm 安装会中断）；改为将 `moltbot` 放在 `devDependencies` 或 `peerDependencies` 中（运行时通过 jiti 别名解析 `clawdbot/plugin-sdk`）。
- 从 `https://molt.bot/*` 提供的安装程序：位于兄弟仓库 `../molt.bot`（`public/install.sh`、`public/install-cli.sh`、`public/install.ps1`）。
- 消息频道：在重构共享逻辑时，始终考虑**所有**内置+扩展频道（路由、允许列表、配对、命令门控、入门、文档）。
  - 核心频道文档：`docs/channels/`
  - 核心频道代码：`src/telegram`、`src/discord`、`src/slack`、`src/signal`、`src/imessage`、`src/web`（WhatsApp web）、`src/channels`、`src/routing`
  - 扩展（频道插件）：`extensions/*`（例如 `extensions/msteams`、`extensions/matrix`、`extensions/zalo`、`extensions/zalouser`、`extensions/voice-call`）
- 添加频道/扩展/应用/文档时，查看 `.github/labeler.yml` 以了解标签覆盖范围。

## 文档链接（Mintlify）
- 文档托管在 Mintlify（docs.molt.bot）上。
- `docs/**/*.md` 中的内部文档链接：根目录相对路径，无 `.md`/`.mdx`（例如：`[Config](/configuration)`）。
- 章节交叉引用：在根目录相对路径上使用锚点（例如：`[Hooks](/configuration#hooks)`）。
- 文档标题和锚点：避免在标题中使用破折号和撇号，因为它们会破坏 Mintlify 锚点链接。
- 当 Peter 请求链接时，用完整的 `https://docs.molt.bot/...` URL 回复（而不是根目录相对路径）。
- 当你接触文档时，在回复结束时附上你引用的 `https://docs.molt.bot/...` URL。
- README（GitHub）：保持绝对文档 URL（`https://docs.molt.bot/...`），以便链接在 GitHub 上正常工作。
- 文档内容必须是通用的：没有个人设备名称/主机名/路径；使用占位符如 `user@gateway-host` 和"网关主机"。

## exe.dev VM 操作（一般）
- 访问：稳定路径是 `ssh exe.dev` 然后 `ssh vm-name`（假设 SSH 密钥已设置）。
- SSH 不稳定：使用 exe.dev Web 终端或 Shelley（Web agent）；为长时间操作保持 tmux 会话。
- 更新：`sudo npm i -g moltbot@latest`（全局安装需要在 `/usr/lib/node_modules` 上使用 root）。
- 配置：使用 `moltbot config set ...`；确保设置了 `gateway.mode=local`。
- Discord：仅存储原始令牌（没有 `DISCORD_BOT_TOKEN=` 前缀）。
- 重启：停止旧的网关并运行：
  `pkill -9 -f moltbot-gateway || true; nohup moltbot gateway run --bind loopback --port 18789 --force > /tmp/moltbot-gateway.log 2>&1 &`
- 验证：`moltbot channels status --probe`、`ss -ltnp | rg 18789`、`tail -n 120 /tmp/moltbot-gateway.log`。

## 构建、测试和开发命令
- 运行时基线：Node **22+**（保持 Node + Bun 路径正常工作）。
- 安装依赖：`pnpm install`
- 提交前钩子：`prek install`（运行与 CI 相同的检查）
- 同时支持：`bun install`（在接触依赖/补丁时保持 `pnpm-lock.yaml` + Bun 补丁同步）。
- 对于 TypeScript 执行（脚本、开发、测试）首选 Bun：`bun <file.ts>` / `bunx <tool>`。
- 在开发中运行 CLI：`pnpm moltbot ...`（bun）或 `pnpm dev`。
- Node 仍支持运行构建的输出（`dist/*`）和生产安装。
- Mac 打包（开发）：`scripts/package-mac-app.sh` 默认为当前架构。发布清单：`docs/platforms/mac/release.md`。
- 类型检查/构建：`pnpm build`（tsc）
- Lint/格式化：`pnpm lint`（oxlint）、`pnpm format`（oxfmt）
- 测试：`pnpm test`（vitest）；覆盖率：`pnpm test:coverage`

## 编码风格和命名约定
- 语言：TypeScript（ESM）。首选严格类型；避免 `any`。
- 通过 Oxlint 和 Oxfmt 进行格式化/linting；在提交之前运行 `pnpm lint`。
- 为棘手或非显而易见的逻辑添加简短的代码注释。
- 保持文件简洁；提取辅助函数而不是"V2"副本。对 CLI 选项和通过 `createDefaultDeps` 进行依赖注入使用现有模式。
- 目标是保持文件在约 700 LOC 以下；这只是一个指导原则（不是硬性限制）。在提高清晰度或可测试性时进行拆分/重构。
- 命名：对产品/应用/文档标题使用 **Moltbot**；对 CLI 命令、包/二进制文件、路径和配置键使用 `moltbot`。

## 发布频道（命名）
- stable：仅标记的发布（例如 `vYYYY.M.D`），npm dist-tag `latest`。
- beta：预发布标签 `vYYYY.M.D-beta.N`，npm dist-tag `beta`（可能没有 macOS 应用）。
- dev：在 `main` 上的移动头部（无标签；git checkout main）。

## 测试指南
- 框架：Vitest，带有 V8 覆盖率阈值（70% 行/分支/函数/语句）。
- 命名：使用 `*.test.ts` 匹配源名称；e2e 在 `*.e2e.test.ts` 中。
- 在接触逻辑时推送之前运行 `pnpm test`（或 `pnpm test:coverage`）。
- 不要将测试工作线程设置在 16 以上；已经尝试过了。
- 实时测试（真实密钥）：`CLAWDBOT_LIVE_TEST=1 pnpm test:live`（仅 Moltbot）或 `LIVE=1 pnpm test:live`（包括提供者实时测试）。Docker：`pnpm test:docker:live-models`、`pnpm test:docker:live-gateway`。入门 Docker E2E：`pnpm test:docker:onboard`。
- 完整工具包 + 覆盖内容：`docs/testing.md`。
- 纯测试添加/修复通常**不需要**更改日志条目，除非它们改变了面向用户的行为或用户要求。
- 移动设备：在使用模拟器之前，检查连接的真实设备（iOS + Android），并在可用时优先使用它们。

## 提交和拉取请求指南
- 使用 `scripts/committer "<msg>" <file...>` 创建提交；避免手动 `git add`/`git commit`，以便暂存保持范围限定。
- 遵循简洁、以行动为导向的提交消息（例如，`CLI: add verbose flag to send`）。
- 对相关更改进行分组；避免捆绑不相关的重构。
- 更改日志工作流：将最新发布的版本保持在顶部（没有 `Unreleased`）；发布后，增加版本并开始新的顶部章节。
- PR 应该总结范围，注意执行的测试，并提及任何面向用户的更改或新标志。
- PR 审查流程：当获得 PR 链接时，通过 `gh pr view`/`gh pr diff` 审查，并且**不要**更改分支。
- PR 审查调用：首选单个 `gh pr view --json ...` 来批量获取元数据/评论；仅在需要时运行 `gh pr diff`。
- 在开始审查之前粘贴了 GH Issue/PR 时：运行 `git pull`；如果有本地更改或未推送的提交，请在审查之前停止并警告用户。
- 目标：合并 PR。当提交干净时首选 **rebase**；当历史混乱时首选 **squash**。
- PR 合并流程：从 `main` 创建一个临时分支，将 PR 分支合并到其中（除非提交历史很重要，否则首选 squash；当重要时使用 rebase/merge）。始终尝试合并 PR，除非真的很难，然后使用另一种方法。如果我们 squash，将 PR 作者添加为共同贡献者。应用修复，添加更改日志条目（包括 PR # + 感谢），在最终提交之前运行完整门控，提交，合并回 `main`，删除临时分支，最后在 `main` 上结束。
- 如果你审查 PR 并随后对其进行工作，则通过 merge/squat 合并（没有直接主提交），并始终将 PR 作者添加为共同贡献者。
- 处理 PR 时：在更改日志条目中添加 PR 编号并感谢贡献者。
- 处理 issue 时：在更改日志条目中引用该 issue。
- 合并 PR 时：留下一个 PR 评论，准确解释我们所做的工作并包括 SHA 哈希值。
- 合并来自新贡献者的 PR 时：将其头像添加到 README"感谢所有 clawtributors"缩略图列表中。
- 合并 PR 后：如果缺少贡献者，运行 `bun scripts/update-clawtributors.ts`，然后提交重新生成的 README。

## 简写命令
- `sync`：如果工作树脏，提交所有更改（选择一个合理的约定提交消息），然后 `git pull --rebase`；如果 rebase 冲突且无法解决，则停止；否则 `git push`。

### PR 工作流程（审查 vs 着陆）
- **审查模式（仅 PR 链接）：** 阅读 `gh pr view/diff`；**不要**切换分支；**不要**更改代码。
- **着陆模式：** 从 `main` 创建集成分支，引入 PR 提交（**首选 rebase** 以获得线性历史；当复杂性/冲突使其更安全时**允许合并**），应用修复，添加更改日志（+ 感谢 + PR #），在提交之前在本地运行完整门控（`pnpm lint && pnpm build && pnpm test`），提交，合并回 `main`，然后 `git switch main`（着陆后永远不要停留在主题分支上）。重要：贡献者需要在此之后位于 git 图中！

## 安全和配置提示
- Web 提供者将凭据存储在 `~/.clawdbot/credentials/` 中；如果注销，重新运行 `moltbot login`。
- Pi 会话默认位于 `~/.clawdbot/sessions/` 下；基本目录不可配置。
- 环境变量：参见 `~/.profile`。
- 永远不要提交或发布真实的电话号码、视频或实时配置值。在文档、测试和示例中使用明显虚假的占位符。
- 发布流程：在进行任何发布工作之前，始终阅读 `docs/reference/RELEASING.md` 和 `docs/platforms/mac/release.md`；一旦这些文档回答了它们，就不要询问常规问题。

## 故障排除
- 重新品牌/迁移问题或遗留配置/服务警告：运行 `moltbot doctor`（参见 `docs/gateway/doctor.md`）。

## Agent 特定说明
- 词汇表："makeup" = "mac app"。
- 永远不要编辑 `node_modules`（全局/Homebrew/npm/git 安装也会）。更新会覆盖。技能说明放在 `tools.md` 或 `AGENTS.md` 中。
- Signal："update fly" => `fly ssh console -a flawd-bot -C "bash -lc 'cd /data/clawd/moltbot && git pull --rebase origin main'"` 然后 `fly machines restart e825232f34d058 -a flawd-bot`。
- 处理 GitHub Issue 或 PR 时，在任务结束时打印完整的 URL。
- 回答问题时，仅用高置信度的答案回应：在代码中验证；不要猜测。
- 永远不要更新 Carbon 依赖项。
- 任何具有 `pnpm.patchedDependencies` 的依赖项必须使用精确版本（没有 `^`/`~`）。
- 修补依赖项（pnpm 补丁、覆盖或供应商更改）需要明确的批准；默认情况下不要这样做。
- CLI 进度：使用 `src/cli/progress.ts`（`osc-progress` + `@clack/prompts` 微调器）；不要自己动手制作微调器/条。
- 状态输出：保持表格 + ANSI 安全包装（`src/terminal/table.ts`）；`status --all` = 只读/可粘贴，`status --deep` = 探测。
- 网关当前仅作为菜单栏应用程序运行；没有安装单独的 LaunchAgent/助手标签。通过 Moltbot Mac 应用程序或 `scripts/restart-mac.sh` 重启；要验证/杀死，使用 `launchctl print gui/$UID | grep moltbot` 而不是假设固定标签。**在 macOS 上调试时，通过应用程序启动/停止网关，而不是临时 tmux 会话；在交接之前杀死任何临时隧道。**
- macOS 日志：使用 `./scripts/clawlog.sh` 查询 Moltbot 子系统的统一日志；它支持 follow/tail/category 过滤器，并期望 `/usr/bin/log` 有无密码 sudo。
- 如果本地有共享的护栏，请查看它们；否则遵循此仓库的指导。
- SwiftUI 状态管理（iOS/macOS）：优先使用 `Observation` 框架（`@Observable`、`@Bindable`）而不是 `ObservableObject`/`@StateObject`；除非需要兼容性，否则不要引入新的 `ObservableObject`，并且在接触相关代码时迁移现有用法。
- 连接提供者：添加新连接时，更新每个 UI 表面和文档（macOS 应用程序、web UI、移动设备（如适用）、入门/概述文档），并添加匹配的状态 + 配置表单，以便提供者列表和设置保持同步。
- 版本位置：`package.json`（CLI）、`apps/android/app/build.gradle.kts`（versionName/versionCode）、`apps/ios/Sources/Info.plist` + `apps/ios/Tests/Info.plist`（CFBundleShortVersionString/CFBundleVersion）、`apps/macos/Sources/Moltbot/Resources/Info.plist`（CFBundleShortVersionString/CFBundleVersion）、`docs/install/updating.md`（固定的 npm 版本）、`docs/platforms/mac/release.md`（APP_VERSION/APP_BUILD 示例）、Peekaboo Xcode 项目/Info.plists（MARKETING_VERSION/CURRENT_PROJECT_VERSION）。
- **重启应用程序：**"重启 iOS/Android 应用程序"意味着重新构建（重新编译/安装）并重新启动，而不仅仅是杀死/启动。
- **设备检查：**在测试之前，在触及模拟器/模拟器之前验证连接的真实设备（iOS/Android）。
- iOS Team ID 查找：`security find-identity -p codesigning -v` → 使用 Apple Development（…）TEAMID。回退：`defaults read com.apple.dt.Xcode IDEProvisioningTeamIdentifiers`。
- A2UI 包哈希：`src/canvas-host/a2ui/.bundle.hash` 是自动生成的；忽略意外的更改，仅在需要时通过 `pnpm canvas:a2ui:bundle`（或 `scripts/bundle-a2ui.sh`）重新生成。将哈希作为单独的提交提交。
- 发布签名/公证密钥在 repo 之外管理；遵循内部发布文档。
- 公证 auth 环境变量（`APP_STORE_CONNECT_ISSUER_ID`、`APP_STORE_CONNECT_KEY_ID`、`APP_STORE_CONNECT_API_KEY_P8`）预期在你的环境中（根据内部发布文档）。
- **多 agent 安全：****不要**创建/应用/删除 `git stash` 条目，除非明确请求（这包括 `git pull --rebase --autostash`）。假设其他 agent 可能正在工作；保持不相关的 WIP 不受干扰，避免交叉状态更改。
- **多 agent 安全：**当用户说"push"时，你可以 `git pull --rebase` 以集成最新的更改（永远不要丢弃其他 agent 的工作）。当用户说"commit"时，仅限于你的更改。当用户说"commit all"时，按分组块提交所有内容。
- **多 agent 安全：****不要**创建/删除/修改 `git worktree` 检出（或编辑 `.worktrees/*`），除非明确请求。
- **多 agent 安全：****不要**切换分支/签出不同的分支，除非明确请求。
- **多 agent 安全：**运行多个 agent 是可以的，只要每个 agent 都有自己的会话。
- **多 agent 安全：**当你看到无法识别的文件时，继续进行；专注于你的更改并仅提交这些更改。
- Lint/格式化波动：
  - 如果暂存+未暂存的差异仅是格式化，则自动解决而不询问。
  - 如果已经请求了 commit/push，则自动暂存并将仅格式化的后续操作包括在同一提交中（或如果需要，使用一个小的后续提交），无需额外确认。
  - 仅当更改是语义性的（逻辑/数据/行为）时才询问。
- Lobster 接缝：使用 `src/terminal/palette.ts` 中的共享 CLI 调色板（没有硬编码颜色）；根据需要将调色板应用于入门/配置提示和其他 TTY UI 输出。
- **多 agent 安全：**将报告集中在你的编辑上；避免护栏免责声明，除非真的被阻止；当多个 agent 触及同一个文件时，如果安全则继续；仅在相关时以简短的"存在其他文件"注释结束。
- Bug 调查：在得出结论之前阅读相关 npm 依赖项的源代码和所有相关的本地代码；以高置信度的根本原因为目标。
- 代码风格：为棘手的逻辑添加简短注释；尽可能保持文件在 500 LOC 以下（根据需要拆分/重构）。
- 工具架构护栏（google-antigravity）：在工具输入架构中避免 `Type.Union`；没有 `anyOf`/`oneOf`/`allOf`。对字符串列表使用 `stringEnum`/`optionalStringEnum`（Type.Unsafe 枚举），对 `... | null` 使用 `Type.Optional(...)`。将顶级工具架构保持为 `type: "object"` 和 `properties`。
- 工具架构护栏：避免在工具架构中使用原始 `format` 属性名称；某些验证器将 `format` 视为保留关键字并拒绝该架构。
- 当要求打开"session"文件时，打开 `~/.clawdbot/agents/<agentId>/sessions/*.jsonl` 下的 Pi 会话日志（使用系统提示的 Runtime 行中的 `agent=<id>` 值；除非给出特定 ID，否则为最新的），而不是默认的 `sessions.json`。如果需要来自另一台机器的日志，通过 Tailscale SSH 并在那里读取相同的路径。
- 不要通过 SSH 重建 macOS 应用程序；重建必须直接在 Mac 上运行。
- 永远不要向外部消息界面发送流式/部分回复（WhatsApp、Telegram）；只有最终回复应该在那里传递。流式/工具事件可能仍然会进入内部 UI/控制频道。
- 语音唤醒转发提示：
  - 命令模板应保持 `moltbot-mac agent --message "${text}" --thinking low`；`VoiceWakeForwarder` 已经 shell 转义了 `${text}`。不要添加额外的引号。
  - launchd PATH 是最小的；确保应用程序的 launch agent PATH 包括标准系统路径加上你的 pnpm bin（通常是 `$HOME/Library/pnpm`），以便通过 `moltbot-mac` 调用时 `pnpm`/`moltbot` 二进制文件解析。
- 对于包含 `!` 的手动 `moltbot message send` 消息，使用下面提到的 heredoc 模式来避免 Bash 工具的转义。
- 发布护栏：未经操作员明确同意，不要更改版本号；在运行任何 npm publish/发布步骤之前始终请求许可。

## NPM + 1Password（发布/验证）
- 使用 1password 技能；所有 `op` 命令必须在新的 tmux 会话中运行。
- 登录：`eval "$(op signin --account my.1password.com)"`（应用程序已解锁 + 集成已开启）。
- OTP：`op read 'op://Private/Npmjs/one-time password?attribute=otp'`。
- 发布：`npm publish --access public --otp="<otp>"`（从包目录运行）。
- 在没有本地 npmrc 副作用的情况下验证：`npm view <pkg> version --userconfig "$(mktemp)"`。
- 发布后终止 tmux 会话。
