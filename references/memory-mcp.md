# 记忆层（agentmemory MCP）速查

来源：agentmemory MCP 工具族 + memory-discipline。**记忆层是可选增强：MCP 不可用时，所有决策写入 `task_plan.md` 的 Decisions 表，能力等价。**

## 工具速查

| 场景 | 工具 | 关键参数 |
|---|---|---|
| 任务开始查旧记忆 | `memory_smart_search` | `query`（任务主题）、`project`（项目名）、`limit: 5` |
| 决策敲定即存 | `memory_save` | `content`（决策**带原因**）、`concepts`（2-5 个关键词）、`files`（真实路径） |
| 恢复时列会话 | `memory_sessions` | `limit: 20` |
| 恢复时拉细节 | `memory_recall` | `query`（会话 top concepts）、`limit: 10` |
| 用户纠正了我的做法 | `memory_lesson_save` | 走 lesson，不走 memory |
| 重复同类任务前 | `memory_lesson_recall` | 任务类型作 query |

完整工具表（结构化槽、图谱、治理）见 agentmemory 的 agentmemory-mcp-tools 技能；本技能只用上表这一小撮。

## 记忆纪律

- **先搜后存**：非平凡任务的第一个工具调用是项目范围的 `memory_smart_search`。命中省一次重新发现，未命中只花一次调用。
- **即时存**：决策敲定的那一刻就 `memory_save`，内容必须含原因（"选了游标分页，因为 offset 扫描在 10 万行后失效"），不要攒到会话结束批量存——批量存会丢掉理由。
- **该存什么**：带理由的已定决策、调试发现的隐性约束、环境事实。**不该存**：代码里读得出的东西、临时状态、密钥、步骤流水账（hooks 自动记）。
- **会话结束**：不用手动 recap，hooks 自动汇总。RESUME/HANDOFF 里手动保存的只有关键决策。

## 目录边界匹配（防串仓）

`memory_sessions` 返回多个会话时，按 cwd 匹配当前项目必须用**边界检查**，不是前缀匹配：

```
对 /Users/dev/repo-a-staging，当项目是 /Users/dev/repo-a 时：
前缀匹配（错误）→ repo-a-staging 以 repo-a 开头 → 误匹配，恢复错仓库
边界匹配（正确）→ cwd === 项目路径 或 cwd 以 项目路径 + "/" 开头 → 拒绝，选真正的 repo-a 会话
```

实现即：`session.cwd === projectPath || session.cwd.startsWith(projectPath + "/")`。无匹配会话时回退到全局最近的会话，并明确告知用户。

## 无 MCP 降级路径

| 有 MCP 时的动作 | 无 MCP 时的等价动作 |
|---|---|
| 任务开始 memory_smart_search | 无等价（文件不含历史项目记忆时接受此损失） |
| 决策敲定 memory_save | 写入 task_plan.md 的 Decisions 表（含理由） |
| 恢复 memory_sessions + recall | 读 .planning/handoff.md + task_plan.md（通常已足够） |
| 用户纠正存 lesson | 写入 task_plan.md 的 Decisions 表，标注"用户纠正" |

## Claude Code 可选 hooks 集成

若宿主是 Claude Code 且用户愿意配置，可在 `hooks/hooks.json` 注册：

- `PreCompact`：压缩前提醒刷新 progress.md（Claude Code 的 PreCompact 不支持注入 additionalContext，只能在 stdout 打诊断信息）。
- `SessionEnd` / `Stop`：自动执行"缺什么补什么"检查——handoff.md 不存在或 task_plan.md 的 Next Step 落后于实际进度时提醒。

不配 hooks 本技能完全可用（纯指令驱动）；hooks 只是把"AI 记得去做"变成"机制保证发生"。
