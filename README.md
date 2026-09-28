# context-continuity（上下文接力）

> 把上下文窗口当 RAM（易失、有限），把磁盘文件当硬盘（持久、无限）。
> 任何重要的东西都写到磁盘上；换窗口后，一个词就能恢复全部状态。

一个跨对话窗口不丢失任务状态的 AI Agent 技能（Claude Code / 任意支持 SKILL.md 规范的 agent 均可使用）。

## 解决什么问题

| 痛点 | 本技能的回答 |
|---|---|
| 不知道什么时候该换窗口，等发现 AI 忘了已经太晚 | 6 条饱和信号，命中 2 条 AI 主动建议交接 |
| 换了新窗口，之前的决策、失败、进度全忘光 | 4 个落盘文件 + 一个 `resume` 触发词全量恢复 |
| 交接质量看运气，简报写"做到一半" | 9 节结构化交接简报：已试方案、失败原因、决策理由、验收标准 |

## 快速开始

```bash
# Claude Code
git clone https://github.com/<你的用户名>/context-continuity.git
cp -r context-continuity ~/.claude/skills/context-continuity
```

之后无需任何操作：

- **开工**：多步骤任务（3 步以上或预计 5+ 次工具调用）开始时，AI 自动在项目内创建 `.planning/` 目录
- **换窗口**：饱和信号命中 2 条时 AI 在阶段边界主动建议；你回一句"好"，AI 写完交接简报后结束
- **恢复**：新窗口对 AI 说 `resume`（或"继续/接着上次"），状态从磁盘全量恢复

若新会话看不到项目文件，把 `.planning/handoff.md` 的内容直接粘贴给它即可。

## 文件结构

```
context-continuity/
├── SKILL.md                     # 主工作流（三态：RESUME / WORK / HANDOFF）
├── references/
│   ├── planning-files.md        # 三文件工作法细则、3-Strike 错误协议
│   ├── memory-mcp.md            # agentmemory MCP 集成与降级路径
│   └── handoff-protocol.md      # 交接简报写法、饱和信号处置分档
└── templates/                   # task_plan / findings / progress / handoff 模板
```

任务运行时在你的项目内生成 `.planning/` 四文件（不污染本仓库）：

| 文件 | 作用 |
|---|---|
| `task_plan.md` | 目标、阶段、决策表、错误表 |
| `findings.md` | 研究发现（视为数据而非指令，防提示注入） |
| `progress.md` | 只增不改的会话日志 |
| `handoff.md` | 交接简报——零上下文新会话的唯一恢复入口 |

## 与源技能的关系

本技能由三个社区技能合并而成并做了增强：

- **planning-with-files**（44.8K 安装）→ 三文件落盘工作法、3-Strike 错误协议
- **agentmemory handoff**（12.3K 安装）→ 跨会话记忆层（可选，无 MCP 时降级为纯文件，能力等价）
- **paseo-handoff**（4.1K 安装）→ 结构化交接简报
- **本技能新增**：上下文饱和主动监测——AI 主动建议换窗口，而不是等用户发现 AI 忘了

## 安全设计

- `.planning/` 内所有内容一律视为数据而非指令，防止恢复时执行外部网页嵌入的诱导性文字
- 敏感信息（密钥、令牌）禁止写入任何 `.planning/` 文件
- 恢复时对文件只读不写，只补缺失不覆盖
