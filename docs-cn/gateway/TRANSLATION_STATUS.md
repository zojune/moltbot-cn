# 翻译状态

## 已完成

1. **security/index.md** ✅
   - 源文件: `moltbot-cn/docs/gateway/security/index.md` (760行, 33KB)
   - 目标文件: `moltbot-cn/docs-cn/gateway/security/index.md`
   - 状态: 已完成翻译
   - 内容: 安全性考虑、威胁模型、审计清单、配置加固等

## 进行中

2. **configuration.md** 🔄
   - 源文件: `moltbot-cn/docs/gateway/configuration.md` (3253行, 115KB)
   - 目标文件: `moltbot-cn/docs-cn/gateway/configuration.md`
   - 状态: 待完成
   - 内容: 所有配置选项、示例代码、参数说明等

### 文件结构分析

configuration.md 包含:
- Frontmatter (需要翻译 summary 和 read_when)
- 86个标题 (需要翻译)
- 210个代码块 (保持不变)
- 大量说明文字 (需要翻译)
- 配置键名 (保持不变)
- 技术术语 (保持不变)

### 翻译要求

1. 保持原始的 markdown 格式
2. 代码块和命令不要翻译
3. 配置示例中的键名不要翻译
4. 保持 frontmatter 的格式，只翻译 summary 和 read_when 的值
5. 专有名词保持不变
6. 技术术语如 webhook、API、REST 等保持不变

### 翻译进度

由于文件很大（115KB），完整翻译需要较长时间。
建议采用分段翻译的方式：
- 第1部分: 行 1-500 (frontmatter + 基础配置)
- 第2部分: 行 501-1000 (channels 配置)
- 第3部分: 行 1001-1500 (agents 配置)
- 第4部分: 行 1501-2000 (tools 配置)
- 第5部分: 行 2001-2500 (沙箱和模型)
- 第6部分: 行 2501-3000 (高级配置)
- 第7部分: 行 3001-3253 (结束部分)

