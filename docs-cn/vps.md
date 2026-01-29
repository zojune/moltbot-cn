---
summary: "VPS hosting hub for Moltbot (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when: 
  - You want to run the Gateway in the cloud
  - You need a quick map of VPS/hosting guides
---
# VPS hosting

This hub links to the supported VPS/hosting 指南 and explains how cloud
deployments work at a high level.

## Pick a 提供商

- **Railway** (one‑click + browser 设置): [Railway](/railway)
- **Northflank** (one‑click + browser 设置): [Northflank](/northflank)
- **Oracle Cloud (Always Free)**: [Oracle](/platforms/oracle) — $0/month (Always Free, ARM; capacity/signup can be finicky)
- **Fly.io**: [Fly.io](/platforms/fly)
- **Hetzner (Docker)**: [Hetzner](/platforms/hetzner)
- **GCP (Compute Engine)**: [GCP](/platforms/gcp)
- **exe.dev** (VM + HTTPS proxy): [exe.dev](/platforms/exe-dev)
- **AWS (EC2/Lightsail/free tier)**: works well too. Video 指南:
  https://x.com/techfrenAJ/状态/2014934471095812547

## How cloud setups work

- The **Gateway runs on the VPS** and owns 状态 + 工作空间.
- You connect from your laptop/phone via the **Control UI** or **Tailscale/SSH**.
- Treat the VPS as the source of truth and **back up** the 状态 + 工作空间.
- Secure 默认: keep the Gateway on loopback and access it via SSH tunnel or Tailscale Serve.
  If you bind to `lan`/`tailnet`, require `gateway.auth.token` or `gateway.auth.password`.

Remote access: [Gateway remote](/Gateway/remote)  
Platforms hub: [Platforms](/platforms)

## Using 节点 with a VPS

You can keep the Gateway in the cloud and pair **节点** on your local devices
(Mac/iOS/Android/headless). Nodes provide local screen/camera/canvas and `system.run`
capabilities while the Gateway stays in the cloud.

Docs: [节点](/节点), [节点 CLI](/cli/节点)
