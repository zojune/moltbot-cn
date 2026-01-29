---
summary: "Moltbot 在 DigitalOcean 上（简单的付费 VPS 选项）"
read_when:
  - 在 DigitalOcean 上设置 Moltbot
  - 为 Moltbot 寻找便宜的 VPS 托管
---

# Moltbot 在 DigitalOcean 上

## 目标

在 DigitalOcean 上运行持久 Moltbot 网关，费用为 **6 美元/月**（或使用预留定价 4 美元/月）。

如果您想要 0 美元/月的选项且不介意 ARM + 特定于提供商的设置，请参阅 [Oracle Cloud 指南](/platforms/oracle)。

## 成本比较（2026 年）

| 提供商 | 方案 | 规格 | 价格/月 | 备注 |
|----------|------|-------|----------|-------|
| Oracle Cloud | 永久免费 ARM | 高达 4 OCPU，24GB RAM | $0 | ARM，容量有限/注册问题 |
| Hetzner | CX22 | 2 vCPU，4GB RAM | €3.79（~$4） | 最便宜的付费选项 |
| DigitalOcean | Basic | 1 vCPU，1GB RAM | $6 | 易用的 UI，良好的文档 |
| Vultr | Cloud Compute | 1 vCPU，1GB RAM | $6 | 多个位置 |
| Linode | Nanode | 1 vCPU，1GB RAM | $5 | 现为 Akamai 的一部分 |

**选择提供商：**
- DigitalOcean：最简单的 UX + 可预测的设置（本指南）
- Hetzner：良好的性价比（参见 [Hetzner 指南](/platforms/hetzner)）
- Oracle Cloud：可以是 0 美元/月，但比较挑剔且仅限 ARM（参见 [Oracle 指南](/platforms/oracle)）

---

## 前提条件

- DigitalOcean 账户（[注册赠送 200 美元免费额度](https://m.do.co/c/signup)）
- SSH 密钥对（或愿意使用密码认证）
- 约 20 分钟

## 1) 创建 Droplet

1. 登录 [DigitalOcean](https://cloud.digitalocean.com/)
2. 点击 **Create → Droplets**
3. 选择：
   - **区域：** 离您最近的位置（或您的用户）
   - **镜像：** Ubuntu 24.04 LTS
   - **大小：** Basic → Regular → **$6/月**（1 vCPU，1GB RAM，25GB SSD）
   - **认证：** SSH 密钥（推荐）或密码
4. 点击 **Create Droplet**
5. 记下 IP 地址

## 2) 通过 SSH 连接

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) 安装 Moltbot

```bash
# 更新系统
apt update && apt upgrade -y

# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# 安装 Moltbot
curl -fsSL https://molt.bot/install.sh | bash

# 验证
moltbot --version
```

## 4) 运行入门向导

```bash
moltbot onboard --install-daemon
```

向导将引导您完成：
- 模型认证（API 密钥或 OAuth）
- 频道设置（Telegram、WhatsApp、Discord 等）
- 网关令牌（自动生成）
- 守护进程安装（systemd）

## 5) 验证网关

```bash
# 检查状态
moltbot status

# 检查服务
systemctl --user status moltbot-gateway.service

# 查看日志
journalctl --user -u moltbot-gateway.service -f
```

## 6) 访问仪表板

网关默认绑定到环回地址。要访问控制 UI：

**选项 A：SSH 隧道（推荐）**
```bash
# 从您的本地机器
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# 然后打开：http://localhost:18789
```

**选项 B：Tailscale Serve（HTTPS，仅环回）**
```bash
# 在 droplet 上
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# 配置网关使用 Tailscale Serve
moltbot config set gateway.tailscale.mode serve
moltbot gateway restart
```

打开：`https://<magicdns>/`

注意：
- Serve 使网关保持仅环回，并通过 Tailscale 身份标头进行身份验证。
- 如果需要令牌/密码，请设置 `gateway.auth.allowTailscale: false` 或使用 `gateway.auth.mode: "password"`。

**选项 C：Tailnet 绑定（无 Serve）**
```bash
moltbot config set gateway.bind tailnet
moltbot gateway restart
```

打开：`http://<tailscale-ip>:18789`（需要令牌）。

## 7) 连接您的频道

### Telegram
```bash
moltbot pairing list telegram
moltbot pairing approve telegram <CODE>
```

### WhatsApp
```bash
moltbot channels login whatsapp
# 扫描二维码
```

有关其他提供商，请参阅[频道](/channels)。

---

## 1GB RAM 的优化

$6 的 droplet 只有 1GB RAM。为了保持运行顺畅：

### 添加交换空间（推荐）
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### 使用更轻量的模型
如果您遇到 OOM，请考虑：
- 使用基于 API 的模型（Claude、GPT）而不是本地模型
- 将 `agents.defaults.model.primary` 设置为更小的模型

### 监控内存
```bash
free -h
htop
```

---

## 持久性

所有状态位于：
- `~/.clawdbot/` — 配置、凭据、会话数据
- `~/clawd/` — 工作区（SOUL.md、内存等）

这些在重启后仍然存在。定期备份它们：
```bash
tar -czvf moltbot-backup.tar.gz ~/.clawdbot ~/clawd
```

---

## Oracle Cloud 永久免费替代方案

Oracle Cloud 提供**永久免费** ARM 实例，其功能明显比这里任何付费选项更强大 — 0 美元/月。

| 您将获得 | 规格 |
|--------------|-------|
| **4 OCPUs** | ARM Ampere A1 |
| **24GB RAM** | 绰绰有余 |
| **200GB 存储** | 块存储卷 |
| **永久免费** | 无信用卡费用 |

**注意事项：**
- 注册可能比较棘手（如果失败请重试）
- ARM 架构 — 大多数东西都可以工作，但某些二进制文件需要 ARM 构建

有关完整的设置指南，请参阅 [Oracle Cloud](/platforms/oracle)。有关注册提示和故障排除注册过程，请参阅此[社区指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)。

---

## 故障排除

### 网关无法启动
```bash
moltbot gateway status
moltbot doctor --non-interactive
journalctl -u moltbot --no-pager -n 50
```

### 端口已被占用
```bash
lsof -i :18789
kill <PID>
```

### 内存不足
```bash
# 检查内存
free -h

# 添加更多交换空间
# 或升级到 $12/月 droplet（2GB RAM）
```

---

## 另请参阅

- [Hetzner 指南](/platforms/hetzner) — 更便宜、更强大
- [Docker 安装](/install/docker) — 容器化设置
- [Tailscale](/gateway/tailscale) — 安全的远程访问
- [配置](/gateway/configuration) — 完整的配置参考
