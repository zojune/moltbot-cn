---
summary: "macOS 权限持久性 (TCC) 和签名要求"
read_when:
  - 调试缺失或卡住的 macOS 权限提示
  - 打包或签名 macOS 应用
  - 更改捆绑包 ID 或应用安装路径
---
# macOS 权限 (TCC)

macOS 权限授予是脆弱的。TCC 将权限授予与应用的
代码签名、捆绑包标识符和磁盘上的路径相关联。如果这些中的任何一个发生变化，
macOS 会将应用视为新的，可能会删除或隐藏提示。

## 稳定权限的要求
- 相同的路径：从固定位置运行应用（对于 Moltbot，`dist/Moltbot.app`）。
- 相同的捆绑包标识符：更改捆绑包 ID 会创建新的权限标识。
- 签名应用：未签名或临时签名的构建不会持久化权限。
- 一致的签名：使用真正的 Apple Development 或 Developer ID 证书
  以便签名在重建之间保持稳定。

临时签名每次构建都会生成新的标识。macOS 会忘记以前的授予，
并且提示可能会完全消失，直到清除陈旧的条目。

## 提示消失时的恢复检查清单
1. 退出应用。
2. 删除系统设置 -> 隐私与安全性中的应用条目。
3. 从相同的路径重新启动应用并重新授予权限。
4. 如果提示仍未出现，请使用 `tccutil` 重置 TCC 条目并重试。
5. 某些权限仅在完全重新启动 macOS 后才会重新出现。

重置示例（根据需要替换捆绑包 ID）：

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

如果您正在测试权限，请务必使用真实证书进行签名。临时
构建仅适用于权限不重要的快速本地运行。
