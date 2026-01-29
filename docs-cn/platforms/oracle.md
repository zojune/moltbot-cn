---
summary: "Moltbot 在 Oracle Cloud 上（永久免费 ARM）"
read_when:
  - 在 Oracle Cloud 上设置 Moltbot
  - 为 Moltbot 寻找低成本 VPS 托管
  - 想要在小型服务器上 24/7 运行 Moltbot
---

# Moltbot 在 Oracle Cloud 上 (OCI)

## 目标

在 Oracle Cloud 的**永久免费** ARM 层上运行持久 Moltbot 网关。

Oracle 的免费层可以是 Moltbot 的绝佳选择（特别是如果您已经拥有 OCI 帐户），但它需要权衡：
- ARM 架构（大多数东西都可以工作，但有些二进制文件可能仅限 x86）
- 容量和注册可能比较挑剔

## 成本比较（2026 年）

| 提供商 | 方案 | 规格 | 价格/月 | 备注 |
|----------|------|-------|----------|-------|
| Oracle Cloud | 永久免费 ARM | 高达 4 OCPU，24GB RAM | $0 | ARM，容量有限 |
| Hetzner | CX22 | 2 vCPU，4GB RAM | ~ $4 | 最便宜的付费选项 |
| DigitalOcean | Basic | 1 vCPU，1GB RAM | $6 | 易用的 UI，良好的文档 |
| Vultr | Cloud Compute | 1 vCPU，1GB RAM | $6 | 多个位置 |
| Linode | Nanode | 1 vCPU，1GB RAM | $5 | 现为 Akamai 的一部分 |

---

## 前提条件

- Oracle Cloud 帐户（[注册](https://www.oracle.com/cloud/free/)）— 如果遇到问题，请参阅[社区注册指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)
- Tailscale 帐户（在 [tailscale.com](https://tailscale.com) 免费提供）
- 约 30 分钟

## 1) 创建 OCI 实例

1. 登录 [Oracle Cloud Console](https://cloud.oracle.com/)
2. 导航到 **Compute → Instances → Create Instance**
3. 配置：
   - **名称：** `moltbot`
   - **镜像：** Ubuntu 24.04 (aarch64)
   - **形状：** `VM.Standard.A1.Flex` (Ampere ARM)
   - **OCPUs：** 2（或高达 4）
   - **内存：** 12 GB（或高达 24 GB）
   - **启动卷：** 50 GB（高达 200 GB 免费）
   - **SSH 密钥：** 添加您的公钥
4. 点击 **Create**
5. 记下公共 IP 地址

**提示：** 如果实例创建失败并显示"Out of capacity"，请尝试不同的可用性域或稍后重试。免费层容量有限。

## 2) 连接和更新

```bash
# 通过公共 IP 连接
ssh ubuntu@YOUR_PUBLIC_IP

# 更新系统
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**注意：** `build-essential` 是某些依赖项的 ARM 编译所必需的。

## 3) 配置用户和主机名

```bash
# 设置主机名
sudo hostnamectl set-hostname moltbot

# 为 ubuntu 用户设置密码
sudo passwd ubuntu

# 启用 lingering（注销后保持用户服务运行）
sudo loginctl enable-linger ubuntu
```

## 4) 安装 Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=moltbot
```

这启用了 Tailscale SSH，因此您可以从 tailnet 上的任何设备通过 `ssh moltbot` 连接 — 无需公共 IP。

验证：

```bash
tailscale status
```

**从现在开始，通过 Tailscale 连接：** `ssh ubuntu@moltbot`（或使用 Tailscale IP）。

## 5) 安装 Moltbot

```bash
curl -fsSL https://molt.bot/install.sh | bash
source ~/.bashrc
```

当提示"您希望如何孵化您的机器人？"时，选择 **"稍后执行"**。

> 注意：如果您遇到 ARM 原生构建问题，请在寻找 Homebrew 之前从系统软件包（例如 `sudo apt install -y build-essential`）开始。

## 6) 配置网关（环回 + 令牌认证）并启用 Tailscale Serve

使用令牌认证作为默认设置。它是可预测的，避免了任何"不安全认证"控制 UI 标志。

```bash
# 在 VM 上保持网关私有
moltbot config set gateway.bind loopback

# 要求网关 + 控制 UI 进行身份验证
moltbot config set gateway.auth.mode token
moltbot doctor --generate-gateway-token

# 通过 Tailscale Serve 暴露（HTTPS + tailnet 访问）
moltbot config set gateway.tailscale.mode serve
moltbot config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart moltbot-gateway
```

## 7) 验证

```bash
# 检查版本
moltbot --version

# 检查守护进程状态
systemctl --user status moltbot-gateway

# 检查 Tailscale Serve
tailscale serve status

# 测试本地响应
curl http://localhost:18789
```

## 8) 锁定 VCN 安全

现在一切正常，锁定 VCN 以阻止除 Tailscale 之外的所有流量。OCI 的虚拟云网络在网络边缘充当防火墙 — 流量在到达您的实例之前被阻止。

1. 在 OCI Console 中转到 **Networking → Virtual Cloud Networks**
2. 点击您的 VCN → **Security Lists** → Default Security List
3. **删除**所有入口规则，除了：
   - `0.0.0.0/0 UDP 41641`（Tailscale）
4. 保留默认出口规则（允许所有出站）

这会在网络边缘阻止端口 22 上的 SSH、HTTP、HTTPS 和其他所有内容。从现在开始，您只能通过 Tailscale 连接。

---

## 访问控制 UI

从 Tailscale 网络上的任何设备：

```
https://moltbot.<tailnet-name>.ts.net/
```

将 `<tailnet-name>` 替换为您的 tailnet 名称（在 `tailscale status` 中可见）。

无需 SSH 隧道。Tailscale 提供：
- HTTPS 加密（自动证书）
- 通过 Tailscale 身份进行身份验证
- 从 tailnet 上的任何设备访问（笔记本电脑、手机等）

---

## 安全：VCN + Tailscale（推荐基线）

使用 VCN 锁定（仅开放 UDP 41641）且网关绑定到环回，您将获得强大的深度防御：公共流量在网络边缘被阻止，管理访问通过您的 tailnet 进行。

这种设置通常消除了仅为了阻止整个互联网的 SSH 暴力破解而对额外基于主机的防火墙规则的*需要* — 但您仍应保持操作系统更新、运行 `moltbot security audit` 并验证您不会意外地在公共接口上监听。

### 已受保护的内容

| 传统步骤 | 需要？ | 原因 |
|------------------|---------|-----|
| UFW 防火墙 | 不需要 | VCN 在流量到达实例之前阻止 |
| fail2ban | 不需要 | 如果在 VCN 阻止端口 22，则没有暴力破解 |
| sshd 加固 | 不需要 | Tailscale SSH 不使用 sshd |
| 禁用 root 登录 | 不需要 | Tailscale 使用 Tailscale 身份，而不是系统用户 |
| 仅 SSH 密钥认证 | 不需要 | Tailscale 通过您的 tailnet 进行身份验证 |
| IPv6 加固 | 通常不需要 | 取决于您的 VCN/子网设置；验证实际分配/暴露的内容 |

### 仍然推荐

- **凭据权限：** `chmod 700 ~/.clawdbot`
- **安全审计：** `moltbot security audit`
- **系统更新：** 定期运行 `sudo apt update && sudo apt upgrade`
- **监控 Tailscale：** 在 [Tailscale 管理控制台](https://login.tailscale.com/admin) 中审查设备

### 验证安全姿势

```bash
# 确认没有监听公共端口
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# 验证 Tailscale SSH 处于活动状态
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# 可选：完全禁用 sshd
sudo systemctl disable --now ssh
```

---

## 回退：SSH 隧道

如果 Tailscale Serve 不工作，请使用 SSH 隧道：

```bash
# 从您的本地机器（通过 Tailscale）
ssh -L 18789:127.0.0.1:18789 ubuntu@moltbot
```

然后打开 `http://localhost:18789`。

---

## 故障排除

### 实例创建失败（"Out of capacity"）
免费层 ARM 实例很受欢迎。请尝试：
- 不同的可用性域
- 在非高峰时段重试（清晨）
- 选择形状时使用"Always Free"过滤器

### Tailscale 无法连接
```bash
# 检查状态
sudo tailscale status

# 重新认证
sudo tailscale up --ssh --hostname=moltbot --reset
```

### 网关无法启动
```bash
moltbot gateway status
moltbot doctor --non-interactive
journalctl --user -u moltbot-gateway -n 50
```

### 无法访问控制 UI
```bash
# 验证 Tailscale Serve 正在运行
tailscale serve status

# 检查网关是否正在监听
curl http://localhost:18789

# 如果需要，重启
systemctl --user restart moltbot-gateway
```

### ARM 二进制文件问题
某些工具可能没有 ARM 构建。检查：
```bash
uname -m  # 应该显示 aarch64
```

大多数 npm 软件包都可以正常工作。对于二进制文件，请寻找 `linux-arm64` 或 `aarch64` 版本。

---

## 持久性

所有状态位于：
- `~/.clawdbot/` — 配置、凭据、会话数据
- `~/clawd/` — 工作区（SOUL.md、内存、人工制品）

定期备份：

```bash
tar -czvf moltbot-backup.tar.gz ~/.clawdbot ~/clawd
```

---

## 另请参阅

- [网关远程访问](/gateway/remote) — 其他远程访问模式
- [Tailscale 集成](/gateway/tailscale) — 完整的 Tailscale 文档
- [网关配置](/gateway/configuration) — 所有配置选项
- [DigitalOcean 指南](/platforms/digitalocean) — 如果您想要付费 + 更简单的注册
- [Hetzner 指南](/platforms/hetzner) — 基于 Docker 的替代方案
