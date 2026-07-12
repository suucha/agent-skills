# 上下文超长时主动存档讨论要点 — 范围决策

- **Date**: 2026-07-12 14:58
- **Participants**: developer + Claude Code
- **Type**: design

## Background

用户在 agent-memory 技能使用过程中提出一个新需求：当 agent 检测到会话上下文超长时，能否主动把当前讨论要点保存下来（无论是否已经形成决策）。这超出了当前技能"决策驱动存档"的设计哲学，需要判断是否纳入当前版本。

## Discussion points and decisions

### 1. 上下文超长触发存档 — 范围决策

**Problem**: 是否在本版本加入"上下文超长时主动存档讨论要点"的功能？

**Options**:
- Option A: PreCompact 钩子触发 — Claude Code 在自动压缩前会触发 `PreCompact` 事件，挂一个 hook 在那时把当前讨论要点 dump 到 session 文件。最可靠，但需要配置 `settings.json` 的 hook。
- Option B: Agent 自检规则 — 在 SKILL.md 里加一条"对话超过 N 轮 / 感觉上下文变重时主动存档"。简单但阈值难定、容易漏。
- Option C: 保持现状，本版本不加，下个版本再考虑。

**Decision**: Option C — 保持现状，本版本不加入此功能；PreCompact hook 列入下个版本计划。

**Rationale**:
- 当前版本的核心设计哲学是"决策驱动存档"，CLAUDE.md / SKILL.md 已经把决策触发规则强化得很清晰（包括"决定不做某事"也算 first-class decision）
- 加入"上下文长度触发"会引入一个新维度的信号，需要先回答一个设计问题：如何区分"进行中讨论的快照"和"已定决策"，否则会把半成品结论污染决策记录
- 这是个有意义的扩展点，但当前优先级是把现有机制打磨好，避免在没看清取舍前就堆功能
- PreCompact hook 是最可靠的实现路径，下个版本可以直接从这里起步

---

### 2. 实现路径预判（留给下个版本参考）

**Problem**: 真要做时走哪条路？

**Options**:
- PreCompact hook（最可靠）：`settings.json` 配 hook，压缩前 dump 讨论要点
- Agent 自检规则：SKILL.md 里加阈值规则
- 两者结合：hook 兜底 + 规则做主动检查点

**Decision**: 下个版本优先用 PreCompact hook；agent 自检规则作为补充而非主路径。

**Rationale**:
- Hook 是事件驱动的，不依赖 agent 自觉，最不容易漏
- Agent 自检规则的阈值是软规则，LLM 容易 deprioritize（这正是当前决策触发规则要靠 self-check 强制执行的原因）
- 真要做时还需设计：checkpoint 类型的存档如何与 decision 类型在 `index.yaml` 里区分，避免召回时把半成品结论当成定论

## Execution plan

- **本版本**：不做任何改动，保持现状
- **下个版本**：调研 PreCompact hook 接口；设计 checkpoint vs decision 的索引区分；评估是否引入单独的文件类型

## Follow-ups / TODOs

- [ ] 下个版本：调研 PreCompact hook 的接口细节
- [ ] 下个版本：设计 checkpoint 类型存档的索引结构（与 decision 区分）
- [ ] 评估：是否需要单独的文件类型（如 `checkpoint.md`）承载此类存档
