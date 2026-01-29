---
summary: "后台 exec 执行和进程管理"
read_when:
  - 添加或修改后台 exec 行为
  - 调试长时间运行的 exec 任务
---

# 后台 Exec + 进程工具

Moltbot 通过 `exec` 工具运行 shell 命令,并将长时间运行的
任务保存在内存中。`process` 工具管理这些后台会话。

## exec 工具

关键参数:
- `command`(必需)
- `yieldMs`(默认 10000):在此延迟后自动后台化
- `background`(布尔值):立即后台化
- `timeout`(秒,默认 1800):在此超时后终止进程
- `elevated`(布尔值):如果启用/允许提升模式,则在主机上运行
- 需要真正的 TTY?设置 `pty: true`。
- `workdir`、`env`

行为:
- 前台运行直接返回输出。
- 后台化时(显式或超时),工具返回 `status: "running"` + `sessionId` 和简短的尾部输出。
- 输出保存在内存中,直到会话被轮询或清除。
- 如果 `process` 工具不被允许,`exec` 将同步运行并忽略 `yieldMs`/`background`。

## 子进程桥接

在 exec/process 工具之外生成长时间运行的子进程时(例如,CLI 重新生成或 gateway 助手),附加子进程桥接助手,以便终止信号被转发并在退出/错误时分离监听器。这可以避免 systemd 上的孤立进程,并使关闭行为在各平台上保持一致。

环境覆盖:
- `PI_BASH_YIELD_MS`:默认 yield(毫秒)
- `PI_BASH_MAX_OUTPUT_CHARS`:内存输出上限(字符)
- `CLAWDBOT_BASH_PENDING_MAX_OUTPUT_CHARS`:每个流的待处理 stdout/stderr 上限(字符)
- `PI_BASH_JOB_TTL_MS`:已完成会话的 TTL(毫秒,限制为 1m–3h)

配置(首选):
- `tools.exec.backgroundMs`(默认 10000)
- `tools.exec.timeoutSec`(默认 1800)
- `tools.exec.cleanupMs`(默认 1800000)
 - `tools.exec.notifyOnExit`(默认 true):当后台 exec 退出时将系统事件排队 + 请求心跳。

## process 工具

操作:
- `list`:运行中 + 已完成的会话
- `poll`:为会话排出新输出(也报告退出状态)
- `log`:读取聚合输出(支持 `offset` + `limit`)
- `write`:发送 stdin(`data`、可选 `eof`)
- `kill`:终止后台会话
- `clear`:从内存中删除已完成的会话
- `remove`:如果正在运行则终止,如果已完成则清除

注意事项:
- 只有后台化的会话才会被列出/持久化到内存中。
- 进程重启后会话将丢失(无磁盘持久性)。
- 只有在您运行 `process poll/log` 并记录工具结果时,会话日志才会保存到聊天历史中。
- `process` 的作用域是每个代理;它只看到由该代理启动的会话。
- `process list` 包含派生的 `name`(命令动词 + 目标),用于快速扫描。
- `process log` 使用基于行的 `offset`/`limit`(省略 `offset` 以获取最后 N 行)。

## 示例

运行长任务并稍后轮询:
```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```
```json
{"tool": "process", "action": "poll", "sessionId": "<id>"}
```

立即在后台启动:
```json
{"tool": "exec", "command": "npm run build", "background": true}
```

发送 stdin:
```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
