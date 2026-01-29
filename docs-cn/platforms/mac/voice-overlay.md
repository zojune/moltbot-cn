---
summary: "唤醒词和按住说话重叠时的语音叠加生命周期"
read_when:
  - 调整语音叠加行为
---
# macOS 上的语音叠加生命周期

受众：macOS 应用贡献者。目标：当唤醒词和按住说话重叠时保持语音叠加可预测。

### 当前意图
- 如果叠加已通过唤醒词可见并且用户按下热键，热键会话*采用*现有文本而不是重置它。按住热键时叠加保持向上。当用户释放时：如果有修剪的文本则发送，否则关闭。
- 仅唤醒词在静音时仍自动发送；按住说话在释放时立即发送。

### 已实施（2025 年 12 月 9 日）
- 叠加会话现在每次捕获（唤醒词或按住说话）都携带一个令牌。当令牌不匹配时，部分/最终/发送/关闭/级别更新会被删除，从而避免陈旧的回调。
- 按住说话采用任何可见的叠加文本作为前缀（因此当唤醒叠加向上时按下热键会保留文本并附加新语音）。它等待高达 1.5 秒以获得最终转录，然后回退到当前文本。
- Chime/叠加日志在 `voicewake.overlay`、`voicewake.ptt` 和 `voicewake.chime` 类别中以 `info` 发出（会话开始、部分、最终、发送、关闭、chime 原因）。

### 后续步骤
1. **VoiceSessionCoordinator (actor)**
   - 一次只拥有一个 `VoiceSession`。
   - API（基于令牌）：`beginWakeCapture`、`beginPushToTalk`、`updatePartial`、`endCapture`、`cancel`、`applyCooldown`。
   - 删除带有陈旧令牌的回调（防止旧识别器重新打开叠加）。
2. **VoiceSession (model)**
   - 字段：`token`、`source`（wakeWord|pushToTalk）、提交/易失性文本、chime 标志、计时器（自动发送、空闲）、`overlayMode`（显示|编辑|发送）、冷却截止时间。
3. **叠加绑定**
   - `VoiceSessionPublisher` (`ObservableObject`) 将活动会话镜像到 SwiftUI。
   - `VoiceWakeOverlayView` 仅通过发布器渲染；它从不直接变异全局单例。
   - 叠加用户操作（`sendNow`、`dismiss`、`edit`）使用会话令牌回调到协调器。
4. **统一发送路径**
   - 在 `endCapture` 时：如果修剪的文本为空 → 关闭；否则 `performSend(session:)`（播放发送 chime 一次、转发、关闭）。
   - 按住说话：无延迟；唤醒词：自动发送的可选延迟。
   - 在按住说话完成后对唤醒运行时应用短冷却，以便唤醒词不会立即重新触发。
5. **日志记录**
   - 协调器在子系统 `bot.molt`、类别 `voicewake.overlay` 和 `voicewake.chime` 中发出 `.info` 日志。
   - 关键事件：`session_started`、`adopted_by_push_to_talk`、`partial`、`finalized`、`send`、`dismiss`、`cancel`、`cooldown`。

### 调试清单
- 在重现粘性叠加时流式传输日志：

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- 验证只有一个活动会话令牌；协调器应删除陈旧的回调。
- 确保按住说话释放始终使用活动令牌调用 `endCapture`；如果文本为空，预期在没有 chime 或发送的情况下 `dismiss`。

### 迁移步骤（建议）
1. 添加 `VoiceSessionCoordinator`、`VoiceSession` 和 `VoiceSessionPublisher`。
2. 重构 `VoiceWakeRuntime` 以创建/更新/结束会话，而不是直接接触 `VoiceWakeOverlayController`。
3. 重构 `VoicePushToTalk` 以采用现有会话并在释放时调用 `endCapture`；应用运行时冷却。
4. 将 `VoiceWakeOverlayController` 连接到发布器；从运行时/PTT 删除直接调用。
5. 为会话采用、冷却和空文本关闭添加集成测试。
