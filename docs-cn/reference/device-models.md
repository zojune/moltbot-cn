---
summary: "Moltbot 如何为 macOS 应用中的友好名称提供 Apple 设备型号标识符"
read_when:
  - 更新设备型号标识符映射或 NOTICE/许可证文件
  - 更改实例 UI 显示设备名称的方式
---

# 设备型号数据库（友好名称）

macOS 配套应用在**实例**UI 中通过将 Apple 型号标识符（例如 `iPad16,6`、`Mac16,6`）映射到人类可读的名称来显示友好的 Apple 设备型号名称。

映射作为 JSON 提供，位于：

- `apps/macos/Sources/Moltbot/Resources/DeviceModels/`

## 数据源

我们目前从 MIT 许可的仓库提供映射：

- `kyle-seongwoo-jun/apple-device-identifiers`

为了保持构建确定性，JSON 文件固定到特定的上游提交（记录在 `apps/macos/Sources/Moltbot/Resources/DeviceModels/NOTICE.md` 中）。

## 更新数据库

1. 选择您要固定的上游提交（一个用于 iOS，一个用于 macOS）。
2. 更新 `apps/macos/Sources/Moltbot/Resources/DeviceModels/NOTICE.md` 中的提交哈希。
3. 重新下载固定到这些提交的 JSON 文件：

```bash
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/Moltbot/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/Moltbot/Resources/DeviceModels/mac-device-identifiers.json
```

4. 确保 `apps/macos/Sources/Moltbot/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` 仍与上游匹配（如果上游许可证发生更改，请替换它）。
5. 验证 macOS 应用干净地构建（无警告）：

```bash
swift build --package-path apps/macos
```
