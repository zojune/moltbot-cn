---
title: "Node.js + npm（PATH 理性检查）"
summary: "Node.js + npm 安装理性检查：版本、PATH 和全局安装"
read_when:
  - "您安装了 Moltbot 但 `moltbot` 是"命令未找到""
  - "您正在新机器上设置 Node.js/npm"
  - "npm install -g ... 因权限或 PATH 问题而失败"
---

# Node.js + npm（PATH 理性检查）

Moltbot 的运行时基准是 **Node 22+**。

如果您可以运行 `npm install -g moltbot@latest` 但后来看到 `moltbot: command not found`，这几乎总是**PATH**问题：npm 放置全局二进制文件的目录不在您 shell 的 PATH 上。

## 快速诊断

运行：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

如果 `$(npm prefix -g)/bin`（macOS/Linux）或 `$(npm prefix -g)`（Windows）**不在** `echo "$PATH"` 输出中，您的 shell 将无法找到全局 npm 二进制文件（包括 `moltbot`）。

## 修复：将 npm 的全局 bin 目录放在 PATH 上

1) 找到您的全局 npm 前缀：

```bash
npm prefix -g
```

2) 将全局 npm bin 目录添加到您的 shell 启动文件：

- zsh：`~/.zshrc`
- bash：`~/.bashrc`

示例（将路径替换为您的 `npm prefix -g` 输出）：

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

然后打开一个**新终端**（或在 zsh 中运行 `rehash` / 在 bash 中运行 `hash -r`）。

在 Windows 上，将 `npm prefix -g` 的输出添加到您的 PATH。

## 修复：避免 `sudo npm install -g` / 权限错误（Linux）

如果 `npm install -g ...` 因 `EACCES` 而失败，请将 npm 的全局前缀切换到用户可写的目录：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

在您的 shell 启动文件中保留 `export PATH=...` 行。

## 推荐的 Node 安装选项

如果 Node/npm 以以下方式安装，您将遇到最少的意外：

- 保持 Node 更新（22+）
- 使全局 npm bin 目录稳定并在新 shell 中位于 PATH 上

常见选择：

- macOS：Homebrew（`brew install node`）或版本管理器
- Linux：您喜欢的版本管理器，或提供 Node 22+ 的发行版支持的安装
- Windows：官方 Node 安装程序、`winget` 或 Windows Node 版本管理器

如果您使用版本管理器（nvm/fnm/asdf 等），请确保它在您日常使用的 shell（zsh vs bash）中初始化，以便它设置的 PATH 在您运行安装程序时存在。
