# Improvements — Rule Gap Log

This file is the **self-maintaining backlog** for the AI session-tracking system.
Every time a decision was discussed but NOT auto-saved (a "missed save"), the
agent records a gap entry here. Future sessions that want to improve the skill
read this file as their work list.

> **How entries get here**: when the user points out a missed save (e.g. "这条刚才没记下来",
> "you didn't save that"), the current agent — whichever one — appends an entry.
> The user never has to route feedback to a specific "maintainer session".

> **How entries get resolved**: when an entry's gap is fixed (a trigger signal /
> rule was added to cover it), mark it `[resolved]` with the date and a one-line
> note on what changed.

---

## Open

### GAP-001 — Auto-save deprioritized mid-conversation (attention competition)
- **Date found**: 2026-07-10
- **Symptom**: multi-decision conversation produced zero saves; agent completed
  the user's task but treated the save instruction as background noise.
- **Root cause**: save runs mid-conversation, where the active user task
  outcompetes the meta-instruction for attention.
- **Fix applied**: added the "self-check before sending EVERY response"
  checkpoint — a forced end-of-reply check, not just a signal list.
- **Status**: [resolved 2026-07-10] checkpoint mechanism added to SKILL.md / AGENTS.md / CLAUDE.md.

### GAP-002 — Negative decisions ("decide NOT to do something") not recognized
- **Date found**: 2026-07-10
- **Symptom**: user said "保持现状不动" (decided NOT to add a summary field to
  index.yaml); agent confirmed but did not save — the phrase reads as a
  non-event, not a decision.
- **Root cause**: trigger signal list only had *affirmative* decisions
  ("就这样定", "用这个方案", ...). "Decide not to" had no matching category.
- **Fix applied**: added "Decision to NOT do something" as a first-class
  category (保持现状 / 不动了 / 不用加 / 暂不需要 / 先这样吧 / not now / leave it /
  skip this / decided against).
- **Status**: [resolved 2026-07-10] new category added to all three rule files.

### GAP-003 — Decision + task 顺序规则未执行（任务压制元指令）
- **Date found**: 2026-07-11
- **Symptom**: 用户说"那就选1"（决策），然后隐含任务"追加记录"。Agent 直接执行了任务，漏记了"选择方案1"这个决策本身。
- **Root cause**: 
  - Self-check 在任务明确时失效：用户的任务（"追加记录3个决策"）太具体，压制了元指令
  - Agent 没有在执行前问："这一轮用户做决策了吗？"
  - 规则已明确："decision + task → save FIRST, then execute"，但未被执行
  - 这是 GAP-001 的变种：即使有 self-check，当任务非常具体时仍会被压制
- **Fix applied**: 补记了这个决策（决策点 #10）
- **Status**: open - 需要加强 self-check 的优先级或在提示词中增加"用户选择方案"作为明确的决策信号

---

## Resolved

(moved here from Open once a fix lands)

<!-- Entry template — copy this when recording a new gap:
### GAP-NNN — [short title]
- **Date found**: YYYY-MM-DD
- **Symptom**: [what happened — what decision was discussed but not saved]
- **Root cause**: [why the signal wasn't caught — which rule was missing/ambiguous]
- **Fix applied**: [what rule/signal was added, or "pending"]
- **Status**: [open / resolved YYYY-MM-DD — note]
-->
