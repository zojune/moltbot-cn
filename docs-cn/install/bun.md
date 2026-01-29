---
summary: "Bun 工作流（实验性）：安装方法和与 pnpm 相比的注意事项"
read_when:
  - 您想要最快的本地开发循环（bun + watch）
  - 您遇到 Bun 安装/补丁/生命周期脚本问题
---

# Bun（实验性）

目标：使用 **Bun** 运行此仓库（可选，不推荐用于 WhatsApp/Telegram）
而不偏离 pnpm 工作流。

⚠️ **不推荐用于网关运行时**（WhatsApp/Telegram 存在 bug）。生产环境请使用 Node。

## 状态

- Bun 是一个可选的本地运行时，用于直接运行 TypeScript（`bun run …`，`bun --watch …`）。
- `pnpm` 是构建的默认方式，仍然完全支持（并被某些文档工具使用）。
- Bun 无法使用 `pnpm-lock.yaml`，会忽略它。

## 安装

默认方式：

```sh
bun install
```

注意：`bun.lock`/`bun.lockb` 被 gitignore，因此无论哪种方式都不会产生仓库提交。如果您想要*不写入 lockfile*：

```sh
bun install --no-save
```

## 构建 / 测试（Bun）

```sh
bun run build
bun run vitest run
```

## Bun 生命周期脚本（默认被阻止）

除非明确信任（`bun pm untrusted` / `pm trust`），否则 Bun 可能会阻止依赖生命周期脚本。
对于此仓库，通常被阻止的脚本不是必需的：

- `@whiskeysockets/baileys` `preinstall`：检查 Node 主版本 >= 20（我们运行 Node 22+）。
- `protobufjs` `postinstall`：发出关于不兼容版本方案的警告（无构建产物）。

如果您遇到需要这些脚本的实际运行时问题，请明确信任它们：

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## 注意事项

- 某些脚本仍然硬编码使用 pnpm（例如 `docs:build`、`ui:*`、`protocol:check`）。目前请通过 pnpm 运行这些脚本。
