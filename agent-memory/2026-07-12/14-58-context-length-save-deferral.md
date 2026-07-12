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

---

### 3. 加入手动触发的 checkpoint（本版本）

**Problem**: 用户希望本版本就能存"进行中讨论快照"，不必等下个版本的 hook。

**Options**:
- 延后到下个版本和 hook 一起做
- 本版本加手动触发（用户主动说"存个检查点"等），auto-trigger 仍走 hook

**Decision**: 本版本加手动触发；auto-trigger 仍延后到下个版本。

**Rationale**:
- 手动触发实现成本低（主要是 SKILL.md 规则 + 索引约定）
- 填补了"决策驱动存档"和"上下文丢失风险"之间的空白
- 不阻塞下个版本 hook 的工作，两者互补

---

### 4. Checkpoint 的存储与索引 — Option A

**Problem**: checkpoint 怎么存？怎么和 decision 在 index 里区分？

**Options**:
- Option A: 同一 session 文件 + tagged section（`### checkpoint. [topic]`），`index.yaml` 的 subTopics 用 `[checkpoint]` 前缀标记
- Option B: 独立文件类型（如 `checkpoint.md`），index 单独一节

**Decision**: Option A。

**Rationale**:
- 简单、和现有追加模式天然兼容
- 大部分 checkpoint 本来就和 decision 混在同一段对话里，分开存反而割裂上下文
- index 用前缀区分，召回时可以一眼看出哪些是定论、哪些是进行中

---

### 5. Checkpoint 升级为 decision 的语义 — 原地升级

**Problem**: 当 checkpoint 后来收敛成 decision 时怎么处理？

**Options**:
- 原地升级：把 `### checkpoint. xxx` 改写成 `### N. xxx`，补全 Decision/Rationale
- 保留 checkpoint + 新建 decision section 指向它

**Decision**: 原地升级。

**Rationale**:
- git 已经保留了"曾经是 checkpoint"的演化历史，文件内不必双份记录
- 文件保持简洁，避免越拉越长
- `index.yaml` 同步把 `[checkpoint]` 前缀去掉

## Execution plan

- **本版本**：加入用户手动触发的 checkpoint（trigger phrases + `### checkpoint.` section + `[checkpoint]` index 前缀 + 原地升级语义）；auto-trigger 仍不动
- **下个版本**：用 PreCompact hook 把 checkpoint 触发自动化

## Follow-ups / TODOs

- [x] 本版本：实现手动 checkpoint 触发（Option A：同文件 + tagged section）
- [x] 本版本：定义原地升级语义
- [ ] 下个版本：PreCompact hook 自动触发
- [ ] 验证：真实使用中 checkpoint → decision 升级流程是否顺畅
