---
summary: "macOS Skills 设置 UI 和网关支持的状态"
read_when:
  - 更新 macOS Skills 设置 UI
  - 更改技能门控或安装行为
---
# Skills (macOS)

macOS 应用通过网关展示 Moltbot 技能；它不会在本地解析技能。

## 数据源
- `skills.status`（网关）返回所有技能以及资格和缺失要求
  （包括捆绑技能的允许列表块）。
- 要求派生自每个 `SKILL.md` 中的 `metadata.moltbot.requires`。

## 安装操作
- `metadata.moltbot.install` 定义安装选项（brew/node/go/uv）。
- 应用调用 `skills.install` 在网关主机上运行安装程序。
- 当提供多个安装程序时，网关仅显示一个首选安装程序
  （brew（如果可用），否则来自 `skills.install` 的 node manager，默认 npm）。

## Env/API 密钥
- 应用将密钥存储在 `~/.clawdbot/moltbot.json` 中的 `skills.entries.<skillKey>` 下。
- `skills.update` 打补丁 `enabled`、`apiKey` 和 `env`。

## 远程模式
- 安装 + 配置更新发生在网关主机上（而不是本地 Mac）。
