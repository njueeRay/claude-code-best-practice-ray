# 仓库深度调研与协作模式解析

[← 返回 README](../README.md)

## 1. 调研目标与范围

本次调研聚焦两个问题：

1. 这个仓库的核心目的是什么。
2. 它如何组织 Agent 化协作，并支持团队持续沉淀。

调研覆盖了以下高价值区域：

- 根文档：`README.md`、`CLAUDE.md`
- 规则系统：`.claude/rules/`
- 可执行配置：`.claude/agents/`、`.claude/commands/`、`.claude/skills/`、`.claude/settings.json`
- 方法论文档：`best-practice/`、`implementation/`、`reports/`
- 参考工作流：`orchestration-workflow/`、`agent-teams/`、`development-workflows/`

## 2. 仓库核心目的（Core Purpose）

这个仓库不是业务应用仓库，而是一个 Claude Code 的最佳实践参考实现仓库。其核心目标可概括为：

1. 把抽象能力转成可复用的工程资产：将 Subagent、Command、Skill、Hooks、Memory、Settings 等机制，转成可直接参考或复刻的文档和实现。
2. 提供可运行的编排范式：通过 Weather 等端到端示例，展示 Command -> Agent -> Skill 的标准落地路径。
3. 建立持续演化的知识底座：通过 `best-practice/`、`reports/`、`tips/`、`videos/` 形成“规范 + 研究 + 实战经验”的闭环。
4. 支持团队协作与规模化复制：通过 settings 层级、hooks、agent memory、agent teams 和任务分工机制，支撑多人协作。

从 `CLAUDE.md` 的定义看，仓库定位非常明确：它是参考实现（reference implementation），目标是演示配置与协作模式，而非交付单一应用功能。

## 3. 协作模式总览

### 3.1 主轴模式：Command -> Agent -> Skill

仓库最核心的协作模式是分层编排：

1. Command 负责用户入口与流程编排。
2. Agent 负责自治执行与上下文隔离。
3. Skill 负责可复用知识或可复用动作。

该模式在 Weather 工作流中体现为：

- Command：`.claude/commands/weather-orchestrator.md`
- Agent：`.claude/agents/weather-agent.md`
- Agent 预加载 Skill：`.claude/skills/weather-fetcher/SKILL.md`
- 独立调用 Skill：`.claude/skills/weather-svg-creator/SKILL.md`
- 输出目录：`orchestration-workflow/`

其中包含两类 Skill 使用方式：

1. Agent Skill（预加载到 Agent）。
2. 独立 Skill（由 Command 在流程中直接调用）。

这让“数据获取”和“结果渲染”天然解耦，便于替换和扩展。

### 3.2 扩展模式：Agent Teams 并行协作

`implementation/claude-agent-teams-implementation.md` 与 `agent-teams/agent-teams-prompt.md` 体现了多角色协作模式：

- Command Architect
- Agent Engineer
- Skill Designer

协作方法是“并行开发 + 共享任务清单 + 统一数据契约”。

关键价值：

1. 每个成员职责单一，降低冲突。
2. 通过共享任务和接口契约，确保模块可拼接。
3. 形成可复制的团队协作模板，适合中大型任务。

### 3.3 配置治理：Settings、Hooks、Memory

从 `.claude/settings.json`、`reports/claude-global-vs-project-settings.md`、`reports/claude-agent-memory.md` 可以看出，该仓库把“治理能力”放在非常高优先级：

1. 权限分层：`allow`、`ask`、`deny` 明确工具使用边界。
2. Hook 事件驱动：覆盖 PreToolUse、PostToolUse、Stop、SessionStart、TaskCompleted 等，保证执行有反馈、有校验。
3. Memory 分层：支持 user/project/local 级别的 agent memory，兼顾跨项目经验与项目内共享。
4. Settings 层级清晰：global、project、local 与命令行覆盖关系明确，便于团队统一与个人个性化并存。

## 4. 目录结构与职责地图

| 目录 | 职责 | 典型价值 |
| --- | --- | --- |
| `best-practice/` | 机制说明与规则化总结 | 用于对齐概念与字段规范 |
| `implementation/` | 可复刻实现案例 | 提供落地模板而非纸面建议 |
| `reports/` | 深入研究与对比分析 | 回答“为什么这样设计” |
| `tips/` | 高密度经验沉淀 | 提供快速提升策略 |
| `orchestration-workflow/` | 端到端示例 | 直观看到编排模式如何执行 |
| `agent-teams/` | 多 Agent 协作演示 | 体现并行协作与任务分配 |
| `.claude/` | 真实运行配置 | 仓库的“可执行内核” |
| `development-workflows/` | 跨模型和流程策略 | 支撑复杂任务的执行框架 |
| `tutorial/`、`videos/` | 学习与传播材料 | 降低上手门槛，强化可推广性 |

## 5. 关键规则与硬约束

结合 `.claude/rules/` 与 `CLAUDE.md`，可抽取以下硬约束：

1. 文档规范化：Markdown 文档需保持单主题、相对链接、结构化编排（见 `.claude/rules/markdown-docs.md`）。
2. Presentation 变更必须委派：`presentation/index.html` 的改动必须走 `presentation-curator` 代理（见 `.claude/rules/presentation.md`）。
3. Subagent 调用方式：禁止通过 bash 间接调用子代理，应使用 Agent 工具语义。
4. CLAUDE.md 控制长度：强调拆分规则到 `.claude/rules/`，避免单文件过长导致执行效果衰减。
5. 提交策略：提倡细粒度提交，便于审阅、回滚与追踪。

这些约束共同作用，目标是提升“可维护性、可审计性、可复用性”。

## 6. 端到端流程解剖（Weather 示例）

以 `weather-orchestrator` 为例，完整链路是：

1. Command 询问用户偏好（摄氏/华氏）。
2. Command 调用 weather-agent 获取温度。
3. weather-agent 使用预加载的 weather-fetcher 访问 Open-Meteo 并返回标准化温度结果。
4. Command 调用 weather-svg-creator，把温度写入 SVG 与 markdown 输出。

该流程体现的设计思想：

1. 单一职责：获取数据与渲染输出分离。
2. 接口清晰：Agent 返回结构化结果，Skill 只消费输入不重新抓取。
3. 可替换性强：更换数据源或更换展示样式，只需替换局部组件。

## 7. 对我们分支开发的建议（面向 dev）

在当前已建立的双远程模型下（origin=我们的仓库，upstream=原仓库），建议按以下方式推进：

1. 把 `upstream/main` 作为“基线真相源”，周期性同步，避免偏离主仓演进。
2. 所有新增实践先在 `dev` 分支验证，再决定是否合并进 `main`。
3. 在 `docs/` 持续沉淀二次研究：
   - 新增协作策略
   - 本地团队约束
   - 失败案例与修正记录
4. 对高频任务优先封装 Command，对复杂自治任务再引入 Agent，避免过度工程化。
5. 为关键能力建立“最小可运行样例”，保持仓库的教学和验证属性。

## 8. 关键证据文件（建议优先阅读）

1. `README.md`
2. `CLAUDE.md`
3. `.claude/settings.json`
4. `.claude/rules/markdown-docs.md`
5. `.claude/rules/presentation.md`
6. `.claude/commands/weather-orchestrator.md`
7. `.claude/agents/weather-agent.md`
8. `.claude/skills/weather-fetcher/SKILL.md`
9. `.claude/skills/weather-svg-creator/SKILL.md`
10. `orchestration-workflow/orchestration-workflow.md`
11. `implementation/claude-agent-teams-implementation.md`
12. `reports/claude-agent-command-skill.md`

## 9. 结论

该仓库的核心竞争力不在“单个技巧”，而在“可执行的协作系统设计”：

1. 用 Command 承接入口与编排。
2. 用 Agent 承接自治执行与隔离。
3. 用 Skill 承接可复用知识与动作。
4. 用 Settings/Hooks/Memory 承接治理与长期演化。

因此，它更像一个“AI 协作工程方法学仓库”，可作为团队建设 Agent Team 体系的基础模板。
