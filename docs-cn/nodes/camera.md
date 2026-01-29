---
summary: "相机捕获（iOS 节点 + macOS 应用）供代理使用：照片（jpg）和短视频片段（mp4）"
read_when:
  - 在 iOS 节点或 macOS 上添加或修改相机捕获功能
  - 扩展代理可访问的 MEDIA 临时文件工作流
---

# 相机捕获（代理）

Moltbot 支持**相机捕获**用于代理工作流：

- **iOS 节点**（通过网关配对）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。
- **Android 节点**（通过网关配对）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。
- **macOS 应用**（通过网关的节点）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。

所有相机访问都受**用户控制的设置**限制。

## iOS 节点

### 用户设置（默认开启）

- iOS 设置选项卡 → **相机** → **允许相机**（`camera.enabled`）
  - 默认值：**开启**（缺失的键被视为已启用）。
  - 关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

### 命令（通过网关 `node.invoke`）

- `camera.list`
  - 响应负载：
    - `devices`：`{ id, name, position, deviceType }` 数组

- `camera.snap`
  - 参数：
    - `facing`：`front|back`（默认：`front`）
    - `maxWidth`：数字（可选；iOS 节点上默认 `1600`）
    - `quality`：`0..1`（可选；默认 `0.9`）
    - `format`：目前为 `jpg`
    - `delayMs`：数字（可选；默认 `0`）
    - `deviceId`：字符串（可选；来自 `camera.list`）
  - 响应负载：
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`、`height`
  - 负载保护：照片被重新压缩以保持 base64 负载低于 5 MB。

- `camera.clip`
  - 参数：
    - `facing`：`front|back`（默认：`front`）
    - `durationMs`：数字（默认 `3000`，限制最大 `60000`）
    - `includeAudio`：布尔值（默认 `true`）
    - `format`：目前为 `mp4`
    - `deviceId`：字符串（可选；来自 `camera.list`）
  - 响应负载：
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### 前台要求

与 `canvas.*` 类似，iOS 节点仅允许在**前台**执行 `camera.*` 命令。后台调用返回 `NODE_BACKGROUND_UNAVAILABLE`。

### CLI 助手（临时文件 + MEDIA）

获取附件的最简单方法是通过 CLI 助手，它会将解码的媒体写入临时文件并打印 `MEDIA:<path>`。

示例：

```bash
moltbot nodes camera snap --node <id>               # 默认：前置和后置（2 个 MEDIA 输出）
moltbot nodes camera snap --node <id> --facing front
moltbot nodes camera clip --node <id> --duration 3000
moltbot nodes camera clip --node <id> --no-audio
```

注意：
- `nodes camera snap` 默认为**两个**朝向，以便为代理提供两个视角。
- 输出文件是临时的（在操作系统临时目录中），除非你构建自己的包装器。

## Android 节点

### 用户设置（默认开启）

- Android 设置页面 → **相机** → **允许相机**（`camera.enabled`）
  - 默认值：**开启**（缺失的键被视为已启用）。
  - 关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

### 权限

- Android 需要运行时权限：
  - `CAMERA` 用于 `camera.snap` 和 `camera.clip`。
  - `RECORD_AUDIO` 用于 `camera.clip` 当 `includeAudio=true` 时。

如果缺少权限，应用会在可能时提示；如果被拒绝，`camera.*` 请求将以 `*_PERMISSION_REQUIRED` 错误失败。

### 前台要求

与 `canvas.*` 类似，Android 节点仅允许在**前台**执行 `camera.*` 命令。后台调用返回 `NODE_BACKGROUND_UNAVAILABLE`。

### 负载保护

照片被重新压缩以保持 base64 负载低于 5 MB。

## macOS 应用

### 用户设置（默认关闭）

macOS 伴侣应用显示一个复选框：

- **设置 → 常规 → 允许相机**（`moltbot.cameraEnabled`）
  - 默认值：**关闭**
  - 关闭时：相机请求返回"相机被用户禁用"。

### CLI 助手（node invoke）

使用主 `moltbot` CLI 在 macOS 节点上调用相机命令。

示例：

```bash
moltbot nodes camera list --node <id>            # 列出相机 ID
moltbot nodes camera snap --node <id>            # 打印 MEDIA:<path>
moltbot nodes camera snap --node <id> --max-width 1280
moltbot nodes camera snap --node <id> --delay-ms 2000
moltbot nodes camera snap --node <id> --device-id <id>
moltbot nodes camera clip --node <id> --duration 10s          # 打印 MEDIA:<path>
moltbot nodes camera clip --node <id> --duration-ms 3000      # 打印 MEDIA:<path>（旧标志）
moltbot nodes camera clip --node <id> --device-id <id>
moltbot nodes camera clip --node <id> --no-audio
```

注意：
- `moltbot nodes camera snap` 默认为 `maxWidth=1600`，除非被覆盖。
- 在 macOS 上，`camera.snap` 在预热/曝光稳定后等待 `delayMs`（默认 2000ms）然后拍摄。
- 照片负载被重新压缩以保持 base64 低于 5 MB。

## 安全性和实际限制

- 相机和麦克风访问会触发常见的操作系统权限提示（并需要在 Info.plist 中使用说明字符串）。
- 视频片段受到限制（目前 `<= 60s`）以避免过大的节点负载（base64 开销 + 消息限制）。

## macOS 屏幕视频（操作系统级别）

对于*屏幕*视频（而非相机），请使用 macOS 伴侣：

```bash
moltbot nodes screen record --node <id> --duration 10s --fps 15   # 打印 MEDIA:<path>
```

注意：
- 需要 macOS **屏幕录制**权限（TCC）。
