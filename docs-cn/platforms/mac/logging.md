---
summary: "Moltbot 日志记录：滚动诊断文件日志 + 统一日志隐私标志"
read_when:
  - 捕获 macOS 日志或调查私人数据日志记录
  - 调试语音唤醒/会话生命周期问题
---
# 日志记录（macOS）

## 滚动诊断文件日志（调试窗格）
Moltbot 通过 swift-log 路由 macOS 应用日志（默认为统一日志记录），并且可以在您需要持久捕获时向磁盘写入本地、旋转的文件日志。

- 详细程度：**调试窗格 → 日志 → 应用日志记录 → 详细程度**
- 启用：**调试窗格 → 日志 → 应用日志记录 → "写入滚动诊断日志 (JSONL)"**
- 位置：`~/Library/Logs/Moltbot/diagnostics.jsonl`（自动旋转；旧文件后缀为 `.1`、`.2`、...）
- 清除：**调试窗格 → 日志 → 应用日志记录 → "清除"**

注意：
- 这**默认关闭**。仅在主动调试时启用。
- 将文件视为敏感；不要在未经审查的情况下分享它。

## macOS 上的统一日志私人数据

统一日志会编辑大多数有效负载，除非子系统选择加入 `privacy -off`。根据 Peter 关于 macOS [日志记录隐私恶作剧](https://steipete.me/posts/2025/logging-privacy-shenanigans)（2025）的说明，这由 `/Library/Preferences/Logging/Subsystems/` 中按子系统名称键入的 plist 控制。只有新的日志条目才会采用该标志，因此在重现问题之前启用它。

## 为 Moltbot (`bot.molt`) 启用
- 首先将 plist 写入临时文件，然后以 root 身份原子安装它：

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

- 不需要重新启动；logd 会快速注意到文件，但只有新的日志行才会包含私人有效负载。
- 使用现有的帮助程序查看更丰富的输出，例如 `./scripts/clawlog.sh --category WebChat --last 5m`。

## 调试后禁用
- 删除覆盖：`sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`。
- 可选运行 `sudo log config --reload` 以强制 logd 立即删除覆盖。
- 请记住，此表面可以包括电话号码和消息正文；仅在您主动需要额外详细信息时保留 plist。
