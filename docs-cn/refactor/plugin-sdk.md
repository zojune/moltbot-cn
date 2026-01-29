---
summary: "计划：一个干净的插件 SDK + 运行时用于所有消息连接器"
read_when:
  - 定义或重构插件架构
  - 将频道连接器迁移到插件 SDK/运行时
---
# 插件 SDK + 运行时重构计划

目标：每个消息连接器都是使用一个稳定 API 的插件（捆绑或外部）。
没有插件直接从 `src/**` 导入。所有依赖项都通过 SDK 或运行时。

## 为什么现在
- 当前的连接器混合了模式：直接核心导入、仅 dist 桥接和自定义助手。
- 这使得升级变得脆弱，并阻碍了干净的外部插件表面。

## 目标架构（两层）

### 1) 插件 SDK（编译时、稳定、可发布）
范围：类型、助手和配置工具。没有运行时状态，没有副作用。

内容（示例）：
- 类型：`ChannelPlugin`、适配器、`ChannelMeta`、`ChannelCapabilities`、`ChannelDirectoryEntry`。
- 配置助手：`buildChannelConfigSchema`、`setAccountEnabledInConfigSection`、`deleteAccountFromConfigSection`、
  `applyAccountNameToChannelSection`。
- 配对助手：`PAIRING_APPROVED_MESSAGE`、`formatPairingApproveHint`。
- 入门助手：`promptChannelAccessConfig`、`addWildcardAllowFrom`、入门类型。
- 工具参数助手：`createActionGate`、`readStringParam`、`readNumberParam`、`readReactionParams`、`jsonResult`。
- 文档链接助手：`formatDocsLink`。

交付：
- 作为 `@clawdbot/plugin-sdk` 发布（或从核心导出为 `clawdbot/plugin-sdk`）。
- Semver，具有明确的稳定性保证。

### 2) 插件运行时（执行表面、注入）
范围：所有接触核心运行时行为的东西。
通过 `MoltbotPluginApi.runtime` 访问，因此插件从不导入 `src/**`。

提议的表面（最小但完整）：
```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: MoltbotConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: MoltbotConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: { text?: string; mediaUrls?: string[]; mediaUrl?: string }) =>
            void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // Teams 风格流程的适配器
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: "dm" | "group" | "channel"; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: MoltbotConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(cfg: MoltbotConfig, channel: string, accountId: string, groupId: string): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: MoltbotConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: MoltbotConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: MoltbotConfig): string;
  };
};
```

注意事项：
- 运行时是访问核心行为的唯一方式。
- SDK 故意保持小而稳定。
- 每个运行时方法都映射到现有的核心实现（没有重复）。

## 迁移计划（分阶段、安全）

### 第 0 阶段：脚手架
- 引入 `@clawdbot/plugin-sdk`。
- 使用上述表面向 `MoltbotPluginApi` 添加 `api.runtime`。
- 在过渡窗口期间保持现有导入（弃用警告）。

### 第 1 阶段：桥接清理（低风险）
- 用 `api.runtime` 替换每个扩展的 `core-bridge.ts`。
- 首先迁移 BlueBubbles、Zalo、Zalo Personal（已经接近）。
- 删除重复的桥接代码。

### 第 2 阶段：轻度直接导入插件
- 将 Matrix 迁移到 SDK + 运行时。
- 验证入门、目录、组提及逻辑。

### 第 3 阶段：重度直接导入插件
- 迁移 MS Teams（最大的运行时助手集）。
- 确保回复/输入语义与当前行为匹配。

### 第 4 阶段：iMessage 插件化
- 将 iMessage 移动到 `extensions/imessage`。
- 用 `api.runtime` 替换直接核心调用。
- 保持配置键、CLI 行为和文档完整。

### 第 5 阶段：强制执行
- 添加 lint 规则 / CI 检查：没有从 `src/**` 导入 `extensions/**`。
- 添加插件 SDK/版本兼容性检查（运行时 + SDK semver）。

## 兼容性和版本控制
- SDK：semver，已发布，记录的更改。
- 运行时：每个核心发布版本。添加 `api.runtime.version`。
- 插件声明所需的运行时范围（例如，`moltbotRuntime: ">=2026.2.0"`）。

## 测试策略
- 适配器级单元测试（使用真实核心实现运行运行时函数）。
- 每个插件的黄金测试：确保没有行为漂移（路由、配对、允许列表、提及限制）。
- CI 中使用的单个端到端插件示例（安装 + 运行 + 冒烟测试）。

## 未解决的问题
- 在哪里托管 SDK 类型：单独的包还是核心导出？
- 运行时类型分发：在 SDK（仅类型）中还是在核心中？
- 如何为捆绑与外部插件公开文档链接？
- 我们是否允许在过渡期间为存储库中的插件进行有限的直接核心导入？

## 成功标准
- 所有频道连接器都是使用 SDK + 运行时的插件。
- 没有从 `src/**` 导入 `extensions/**`。
- 新连接器模板仅依赖 SDK + 运行时。
- 外部插件可以在没有核心源访问的情况下开发和更新。

相关文档：[插件](/plugin)、[频道](/channels/index)、[配置](/gateway/configuration)。
