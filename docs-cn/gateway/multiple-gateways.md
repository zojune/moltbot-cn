---
summary: "在同一台主机上运行多个 Moltbot Gateway（隔离、端口和配置文件）"
read_when:
  - 在同一台机器上运行多个 Gateway
  - 每个 Gateway 需要隔离的配置/状态/端口
---
# 多个 Gateway（同一主机）

大多数配置应该使用一个 Gateway，因为单个 Gateway 可以处理多个消息连接和 agent。如果您需要更强的隔离或冗余（例如，救援 bot），请使用隔离的配置文件/端口运行独立的 Gateway。

## 隔离检查清单（必需）
- `CLAWDBOT_CONFIG_PATH` — 每个实例的配置文件
- `CLAWDBOT_STATE_DIR` — 每个实例的会话、凭据、缓存
- `agents.defaults.workspace` — 每个实例的工作区根目录
- `gateway.port`（或 `--port`） — 每个实例的唯一端口
- 派生端口（browser/canvas）不得重叠

如果共享这些，您将遇到配置竞争和端口冲突。

## 推荐：配置文件（`--profile`）

配置文件自动限定 `CLAWDBOT_STATE_DIR` + `CLAWDBOT_CONFIG_PATH` 的作用域，并为服务名称添加后缀。

```bash
# 主实例
moltbot --profile main setup
moltbot --profile main gateway --port 18789

# 救援实例
moltbot --profile rescue setup
moltbot --profile rescue gateway --port 19001
```

每个配置文件的服务：
```bash
moltbot --profile main gateway install
moltbot --profile rescue gateway install
```

## 救援 bot 指南

在同一台主机上运行第二个 Gateway，它拥有自己的：
- 配置文件/配置
- 状态目录
- 工作区
- 基础端口（加上派生端口）

这使救援 bot 与主 bot 隔离，以便在主 bot 停机时可以调试或应用配置更改。

端口间距：在基础端口之间至少保留 20 个端口，这样派生的 browser/canvas/CDP 端口永远不会冲突。

### 如何安装（救援 bot）

```bash
# 主 bot（现有或新的，不使用 --profile 参数）
# 运行在端口 18789 + Chrome CDC/Canvas/... 端口
moltbot onboard
moltbot gateway install

# 救援 bot（隔离的配置文件 + 端口）
moltbot --profile rescue onboard
# 注意：
# - 工作区名称默认会加上 -rescue 后缀
# - 端口应该至少是 18789 + 20 个端口，
#   最好选择完全不同的基础端口，如 19789
# - 其余的入门过程与正常情况相同

# 要安装服务（如果在入门期间未自动发生）
moltbot --profile rescue gateway install
```

## 端口映射（派生）

基础端口 = `gateway.port`（或 `CLAWDBOT_GATEWAY_PORT` / `--port`）。

- browser control service port = base + 2（仅 loopback）
- `canvasHost.port = base + 4`
- Browser profile CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配

如果您在配置或环境中覆盖了其中任何一个，必须确保每个实例的唯一性。

## Browser/CDP 注意事项（常见陷阱）

- **不要**在多个实例上将 `browser.cdpUrl` 固定为相同的值。
- 每个实例需要自己的 browser control port 和 CDP 范围（从其 gateway port 派生）。
- 如果需要显式 CDP 端口，请为每个实例设置 `browser.profiles.<name>.cdpPort`。
- 远程 Chrome：使用 `browser.profiles.<name>.cdpUrl`（每个配置文件，每个实例）。

## 手动环境变量示例

```bash
CLAWDBOT_CONFIG_PATH=~/.clawdbot/main.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-main \
moltbot gateway --port 18789

CLAWDBOT_CONFIG_PATH=~/.clawdbot/rescue.json \
CLAWDBOT_STATE_DIR=~/.clawdbot-rescue \
moltbot gateway --port 19001
```

## 快速检查

```bash
moltbot --profile main status
moltbot --profile rescue status
moltbot --profile rescue browser status
```
