---
summary: "Moltbot 在 macOS 上的菜单栏图标状态和动画"
read_when:
  - 更改菜单栏图标行为
---
# 菜单栏图标状态

作者：steipete · 更新时间：2025-12-06 · 范围：macOS 应用 (`apps/macos`)

- **空闲：** 正常的图标动画（眨眼、偶尔摆动）。
- **暂停：** 状态项使用 `appearsDisabled`；没有运动。
- **语音触发（大耳朵）：** 语音唤醒检测器在听到唤醒词时调用 `AppState.triggerVoiceEars(ttl: nil)`，在捕获话语时保持 `earBoostActive=true`。耳朵放大 (1.9倍)，获得圆形耳洞以提高可读性，然后在 1 秒静音后通过 `stopVoiceEars()` 下降。仅从应用内语音管道触发。
- **工作（代理正在运行）：** `AppState.isWorking=true` 驱动"尾巴/腿急跑"微动作：工作时腿部摆动更快，并有轻微偏移。目前在 WebChat 代理运行周围切换；在连接它们时，在周围添加相同的切换。

连接点
- 语音唤醒：运行时测试器在触发时调用 `AppState.triggerVoiceEars(ttl: nil)`，在 1 秒静音后调用 `stopVoiceEars()` 以匹配捕获窗口。
- 代理活动：在工作跨度周围设置 `AppStateStore.shared.setWorking(true/false)`（已在 WebChat 代理调用中完成）。保持跨度短，并在 `defer` 块中重置，以避免卡住的动画。

形状和大小
- 基本图标在 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` 中绘制。
- 耳朵比例默认为 `1.0`；语音提升将 `earScale` 设置为 `1.9` 并切换 `earHoles=true`，而不更改整体帧（18×18 pt 模板图像渲染到 36×36 px Retina 后备存储）。
- 急跑使用高达约 `1.0` 的腿部摆动，并带有轻微的水平抖动；它添加到任何现有的空闲摆动上。

行为说明
- 没有外部 CLI/代理切换耳朵/工作；将其保留在应用自己的信号内部，以避免意外的摆动。
- 保持 TTL 短暂（<10s），以便如果作业挂起，图标会快速返回基线。
