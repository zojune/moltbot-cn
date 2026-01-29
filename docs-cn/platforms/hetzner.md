---
summary: "在廉价的 Hetzner VPS 上 24/7 运行 Moltbot 网关（Docker），具有持久状态和内置二进制文件"
read_when:
  - 您希望在云 VPS（而不是您的笔记本电脑）上 24/7 运行 Moltbot
  - 您希望在自己的 VPS 上拥有生产级、始终在线的网关
  - 您希望完全控制持久性、二进制文件和重启行为
  - 您在 Hetzner 或类似提供商上运行 Docker 中的 Moltbot
---

# Moltbot 在 Hetzner 上（Docker，生产 VPS 指南）

## 目标

使用 Docker 在 Hetzner VPS 上运行持久 Moltbot 网关，具有持久状态、内置二进制文件和安全的重启行为。

如果您想要"约 $5 的 24/7 Moltbot"，这是最简单的可靠设置。
Hetzner 定价会变化；选择最小的 Debian/Ubuntu VPS，如果遇到 OOM 则扩展。

## 我们在做什么（简单术语）？

- 租用一台小型 Linux 服务器（Hetzner VPS）
- 安装 Docker（隔离的应用运行时）
- 在 Docker 中启动 Moltbot 网关
- 在主机上持久化 `~/.clawdbot` + `~/clawd`（在重启/重建中幸存）
- 通过 SSH 隧道从您的笔记本电脑访问控制 UI

可以通过以下方式访问网关：
- 从您的笔记本电脑进行 SSH 端口转发
- 如果您自己管理防火墙和令牌，则直接暴露端口

本指南假设在 Hetzner 上使用 Ubuntu 或 Debian。
如果您在其他 Linux VPS 上，请相应地映射软件包。
有关通用 Docker 流程，请参阅 [Docker](/install/docker)。

---

## 快速路径（经验丰富的操作员）

1) 配置 Hetzner VPS
2) 安装 Docker
3) 克隆 Moltbot 仓库
4) 创建持久主机目录
5) 配置 `.env` 和 `docker-compose.yml`
6) 将所需的二进制文件烘焙到镜像中
7) `docker compose up -d`
8) 验证持久性和网关访问

---

## 您需要什么

- 具有 root 访问权限的 Hetzner VPS
- 从您的笔记本电脑进行 SSH 访问
- 对 SSH + 复制/粘贴的基本了解
- 约 20 分钟
- Docker 和 Docker Compose
- 模型认证凭据
- 可选的提供商凭据
  - WhatsApp QR
  - Telegram 机器人令牌
  - Gmail OAuth

---

## 1) 配置 VPS

在 Hetzner 中创建 Ubuntu 或 Debian VPS。

作为 root 连接：

```bash
ssh root@YOUR_VPS_IP
```

本指南假设 VPS 是有状态的。
不要将其视为一次性基础设施。

---

## 2) 安装 Docker（在 VPS 上）

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

验证：

```bash
docker --version
docker compose version
```

---

## 3) 克隆 Moltbot 仓库

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
```

本指南假设您将构建自定义镜像以保证二进制文件持久性。

---

## 4) 创建持久主机目录

Docker 容器是临时的。
所有长期状态必须位于主机上。

```bash
mkdir -p /root/.clawdbot
mkdir -p /root/clawd

# 将所有权设置为容器用户（uid 1000）：
chown -R 1000:1000 /root/.clawdbot
chown -R 1000:1000 /root/clawd
```

---

## 5) 配置环境变量

在仓库根目录中创建 `.env`。

```bash
CLAWDBOT_IMAGE=moltbot:latest
CLAWDBOT_GATEWAY_TOKEN=change-me-now
CLAWDBOT_GATEWAY_BIND=lan
CLAWDBOT_GATEWAY_PORT=18789

CLAWDBOT_CONFIG_DIR=/root/.clawdbot
CLAWDBOT_WORKSPACE_DIR=/root/clawd

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.clawdbot
```

生成强秘密：

```bash
openssl rand -hex 32
```

**不要提交此文件。**

---

## 6) Docker Compose 配置

创建或更新 `docker-compose.yml`。

```yaml
services:
  moltbot-gateway:
    image: ${CLAWDBOT_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - CLAWDBOT_GATEWAY_BIND=${CLAWDBOT_GATEWAY_BIND}
      - CLAWDBOT_GATEWAY_PORT=${CLAWDBOT_GATEWAY_PORT}
      - CLAWDBOT_GATEWAY_TOKEN=${CLAWDBOT_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${CLAWDBOT_CONFIG_DIR}:/home/node/.clawdbot
      - ${CLAWDBOT_WORKSPACE_DIR}:/home/node/clawd
    ports:
      # 推荐：在 VPS 上保持网关仅环回；通过 SSH 隧道访问。
      # 要公开它，请删除 `127.0.0.1:` 前缀并相应地配置防火墙。
      - "127.0.0.1:${CLAWDBOT_GATEWAY_PORT}:18789"

      # 可选：仅当您针对此 VPS 运行 iOS/Android 节点并且需要 Canvas 主机时。
      # 如果您公开它，请阅读 /gateway/security 并相应地配置防火墙。
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${CLAWDBOT_GATEWAY_BIND}",
        "--port",
        "${CLAWDBOT_GATEWAY_PORT}"
      ]
```

---

## 7) 将所需的二进制文件烘焙到镜像中（关键）

在正在运行的容器中安装二进制文件是一个陷阱。
在运行时安装的任何内容都会在重启时丢失。

技能所需的所​​有外部二进制文件必须在镜像构建时安装。

以下示例仅显示三种常见二进制文件：
- `gog` 用于 Gmail 访问
- `goplaces` 用于 Google Places
- `wacli` 用于 WhatsApp

这些是示例，而不是完整列表。
您可以使用相同的模式安装任意数量的二进制文件。

如果稍后添加依赖于其他二进制文件的新技能，您必须：
1. 更新 Dockerfile
2. 重新构建镜像
3. 重启容器

**示例 Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 示例二进制文件 1：Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 示例二进制文件 2：Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 示例二进制文件 3：WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 使用相同的模式在下面添加更多二进制文件

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

---

## 8) 构建和启动

```bash
docker compose build
docker compose up -d moltbot-gateway
```

验证二进制文件：

```bash
docker compose exec moltbot-gateway which gog
docker compose exec moltbot-gateway which goplaces
docker compose exec moltbot-gateway which wacli
```

预期输出：

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

---

## 9) 验证网关

```bash
docker compose logs -f moltbot-gateway
```

成功：

```
[gateway] listening on ws://0.0.0.0:18789
```

从您的笔记本电脑：

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

打开：

`http://127.0.0.1:18789/`

粘贴您的网关令牌。

---

## 什么持续在哪里（真实来源）

Moltbot 在 Docker 中运行，但 Docker 不是真实来源。
所有长期状态必须在重启、重建和重新启动中幸存。

| 组件 | 位置 | 持久性机制 | 备注 |
|---|---|---|---|
| 网关配置 | `/home/node/.clawdbot/` | 主机卷挂载 | 包括 `moltbot.json`、令牌 |
| 模型认证配置文件 | `/home/node/.clawdbot/` | 主机卷挂载 | OAuth 令牌、API 密钥 |
| 技能配置 | `/home/node/.clawdbot/skills/` | 主机卷挂载 | 技能级状态 |
| 代理工作区 | `/home/node/clawd/` | 主机卷挂载 | 代码和代理人工制品 |
| WhatsApp 会话 | `/home/node/.clawdbot/` | 主机卷挂载 | 保留 QR 登录 |
| Gmail 密钥环 | `/home/node/.clawdbot/` | 主机卷 + 密码 | 需要 `GOG_KEYRING_PASSWORD` |
| 外部二进制文件 | `/usr/local/bin/` | Docker 镜像 | 必须在构建时烘焙 |
| Node 运行时 | 容器文件系统 | Docker 镜像 | 每次镜像构建时重新构建 |
| OS 软件包 | 容器文件系统 | Docker 镜像 | 不要在运行时安装 |
| Docker 容器 | 临时的 | 可重新启动 | 可以安全销毁 |
