---
summary: "`moltbot plugins` CLI 参考(列表、安装、启用/禁用、诊断)"
read_when:
  - 您想安装或管理进程内 Gateway 插件
  - 您想调试插件加载失败
---

# `moltbot plugins`

管理 Gateway 插件/扩展(在进程内加载)。

相关:
- 插件系统: [Plugins](/plugin)
- 插件清单 + 架构: [Plugin manifest](/plugins/manifest)
- 安全加固: [Security](/gateway/security)

## 命令

```bash
moltbot plugins list
moltbot plugins info <id>
moltbot plugins enable <id>
moltbot plugins disable <id>
moltbot plugins doctor
moltbot plugins update <id>
moltbot plugins update --all
```

捆绑的插件随 Moltbot 附带,但默认禁用。使用 `plugins enable` 来激活它们。

所有插件必须附带一个 `moltbot.plugin.json` 文件,其中包含内联 JSON 架构
(`configSchema`,即使为空)。缺失/无效的清单或架构会阻止插件加载并使配置验证失败。

### 安装

```bash
moltbot plugins install <path-or-spec>
```

安全注意事项:像运行代码一样对待插件安装。首选固定版本。

支持的归档文件:`.zip`、`.tgz`、`.tar.gz`、`.tar`。

使用 `--link` 避免复制本地目录(添加到 `plugins.load.paths`):

```bash
moltbot plugins install -l ./my-plugin
```

### 更新

```bash
moltbot plugins update <id>
moltbot plugins update --all
moltbot plugins update <id> --dry-run
```

更新仅适用于从 npm 安装的插件(在 `plugins.installs` 中跟踪)。
