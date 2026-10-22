---
name: agent-memory
description: Auto-saves decisions during conversations and auto-recalls them at session start across agents and machines. Use when user mentions memory setup, saving conversations, checkpoints, or asks for help.
keywords: memory, session, tracking, decisions, recall, knowledge, conversation, notes, remember, 记录, 记忆
---

# Agent Memory

A memory system that turns conversations into **reusable knowledge**: it auto-saves decisions when they happen, and auto-recalls them the next time any agent starts work on the project.

The two mechanisms that make this "memory" and not just an "archive":

1. **Auto-save** — detect decision signals and save *without waiting for the user to ask*
2. **Auto-recall** — read prior decisions at the start of a session and surface them, so no agent ever starts blind

## Why both matter

Saving alone = a diary no one reads. Recalling alone = nothing to recall. The value is the loop: **capture → store → recall → build on prior decisions**. A new agent (or the same agent on a new machine) should be able to pick up where the last one left off.

---

## Usage modes

### Mode 1: Initialize the memory system

When the user says "set up memory" / "设置记忆", "initialize memory" / "初始化记忆", "enable memory" / "启用记忆", or something similar, run the initialization.

**Always create/update both files** (regardless of whether they already exist):
1. **AGENTS.md** (read by opencode)
2. **CLAUDE.md** (read by Claude Code)

Updating both matters because each tool reads only its own config file — if you touch just one, users of the other tool never see the rules.

### Mode 2: Auto-save & auto-recall (ongoing, during every session)

This is the default behavior once initialized — it runs continuously, not on demand.

### Mode 3: On-demand help (when the user asks)

When the user asks what the skill does, how it works, or how to trigger a save — e.g. "memory help", "记忆帮助", "how does memory work", "怎么用记忆", "what does this skill do", "what triggers a save", "怎么才能被记下来" — display the help text in the **"On-demand help text"** section below verbatim (or paraphrased to match the user's language). Do not also run init or save anything; just show the help.

The single highest-value tip in the help is the **affirmative-language** guidance — call it out, because weak language ("选A吧" / "maybe A") is the leading cause of missed saves, and the user can fix it directly by changing how they phrase decisions.

### Mode 4: Manual checkpoint (save in-progress discussion)

When the user wants to capture the current state of a discussion that hasn't yet converged into a decision — e.g. "存个检查点", "记录讨论进度", "checkpoint this", "save a snapshot" — save a **checkpoint**. This is different from Mode 2 auto-save (which fires on decision signals) and from "记一下" (which forces a decision-style save).

Checkpoints capture the *current state of exploration* — options on the table, which way the discussion is leaning, what's still open — so the next session can pick up the thread instead of re-doing the exploration. See **Step 3.5** for the save flow.

---

## On-demand help text

When Mode 3 is triggered, read and display the help text from:

#[[file:references/help-text.md]]

Adapt the language to match the user's (Chinese ↔ English); keep the structure.

---

## Core workflow

### Step 1: Initialize (Mode 1)

#### 0. Upgrade migration — from old `agent-memory/` directory

The storage folder was renamed from `agent-memory/` to `.agent-memory/` (dotfile
convention). Existing installs still have `agent-memory/`. When this skill runs
init / recall and finds `agent-memory/` present but `.agent-memory/` absent,
migrate it so the project converges on the single canonical name.

**Flow (confirm before moving user data — never silent):**

1. Detect: `agent-memory/` exists AND `.agent-memory/` does NOT exist.
2. Tell the user: "检测到旧记忆目录 `agent-memory/`,是否迁移到 `.agent-memory/`?(旧目录会被重命名,历史由 git 保留)"
3. On confirmation, migrate:
   - In a git repo: `git mv agent-memory .agent-memory` (preserves history)
   - Not a git repo: `mv agent-memory .agent-memory`
   - Rewrite references in `AGENTS.md` and `CLAUDE.md`: replace `agent-memory/` → `.agent-memory/` (data-dir references only — do NOT touch `skills/agent-memory`, the skill package path)
4. If BOTH `agent-memory/` and `.agent-memory/` exist: prefer `.agent-memory/`; tell the user the old `agent-memory/` is stale and can be deleted after they confirm nothing is missing.
5. After migration, proceed with `.agent-memory/` for all subsequent save/recall. The recall/save hot path always uses `.agent-memory/` — there is no dual-directory fallback.

**Why confirm first:** moving a user's memory is a data operation. The skill must not silently relocate files; the user confirms, then the one-time move happens.

#### 1.1 Create the directory structure

At the project root, create:

```
.agent-memory/
├── README.md                    # usage instructions
├── IMPROVEMENTS.md              # rule-gap log (self-maintaining backlog of missed saves)
├── template.md                  # session template (copied from this skill)
├── .gitignore                   # exclude sensitive info (optional)
└── YYYY-MM-DD/                  # organized by date
    ├── HH-MM-topic.md
    └── ...
```

Copy these files from the skill's `references/` directory:
- `references/README.md` → `.agent-memory/README.md`
- `references/IMPROVEMENTS.md` → `.agent-memory/IMPROVEMENTS.md`
- `references/template.md` → `.agent-memory/template.md`

#### 1.2 Update AGENTS.md and CLAUDE.md

Append the tracking block to **both** files (see the canonical block in `## Canonical tracking block` below). Check first to avoid duplicates.

### Step 1.5: Session File Structure (strict format)

**Every session file MUST follow this standardized structure.** This enables reliable parsing by any agent.

```markdown
# [Main Topic]

- **Date**: YYYY-MM-DD HH:MM
- **Participants**: [user] + [AI tool name]
- **Type**: [coding / design / discussion / research / other]

## Background
[One paragraph describing why this session happened and what the goal is]

## Discussion points and decisions

### 1. [Decision title]
**Problem**: [One paragraph describing what needs to be solved]
**Options**: [Optional: list of options considered]
**Decision**: [One sentence stating the final choice]
**Rationale**: [Bullet points or paragraph explaining why]

---

### 2. [Next decision title]
**Problem**: ...
**Decision**: ...
**Rationale**: ...

(Continue for all decisions...)

### checkpoint. [In-progress topic / exploration]
**Problem**: [what's being explored — not yet decided]
**Snapshot**: [current state — options on the table, which way leaning, key tradeoffs identified]
**Open questions**: [unresolved questions]

## Execution plan
[Optional: key implementation steps]

## Related content
[Optional: code changes, references]

## Lessons learned
[Optional: reusable patterns, pitfalls to avoid]

## Follow-ups / TODOs
[Optional: pending tasks]
```

**Field names are always English** — `Problem`, `Options`, `Decision`, `Rationale` — regardless of the session's language. Only the content (and `###` section titles) translate. This ensures reliable parsing across languages.

**Critical parsing rules for agents:**
- `### N. [title]` → a **decision point**. MUST have `**Problem:**`, `**Decision:**`, and `**Rationale:**` (Options is optional)
- `### checkpoint. [title]` → an **in-progress discussion snapshot** (not yet a decision). MUST have `**Problem:**`, `**Snapshot:**`, and `**Open questions:**`
- During recall, treat them differently: decisions are settled conclusions; checkpoints are "in-progress, may be superseded" — never present a checkpoint as a settled decision
- Section order is fixed: Background → Discussion → Execution → Related → Lessons → Follow-ups
- Agents extract decisions by:
  1. Reading `index.yaml` → get `subTopics` array (quick overview; entries with `[checkpoint]` prefix are in-progress)
  2. Opening file → locate `## Discussion points and decisions`
  3. Extract all `### [title]` subsections
  4. For each `### N.` subsection, extract Problem/Decision/Rationale
  5. For each `### checkpoint.` subsection, extract Problem/Snapshot/Open questions
  6. Skip low-priority sections (Execution plan, Related content, Follow-ups) during recall

### Step 2: Auto-recall — at the start of a session (Mode 2, ongoing)

**When the conversation begins, or when the user references prior work / a prior decision, ALWAYS do this before responding substantively:**

#### 2.1 Extraction process (two layers)

**Layer 1 — Lightweight scan (always runs, cheap):**
1. Read `.agent-memory/index.yaml` → scan ALL sessions' `subTopics` arrays
2. Judge which sessions are relevant to the current question/task (based on subTopics matching keywords/topics in the user's message)
3. One file read — no deep parsing

**Layer 2 — Deep extraction (only for relevant sessions):**
1. For each session whose subTopics matched, open the file
2. Locate the `## Discussion points and decisions` section
3. Extract all `### [title]` subsections within that section
4. For each subsection, extract:
   - **Problem**: what needed to be solved
   - **Decision**: the final choice
   - **Rationale**: why it was chosen
5. **Skip these sections** (lower recall priority): `## Execution plan`, `## Related content`, `## Follow-ups / TODOs`
6. Also extract key points from `## Lessons learned` if present

**Fallback:** If no sessions match by subTopics, fall back to reading the ~5 most recent sessions in full — better to recall something than nothing.

**Carry context forward** — do not re-litigate settled decisions, do not propose approaches already rejected.

#### 2.2 Why this matters

**This is what makes it memory.** When the user opens a new session or switches machines, the agent already knows what was decided last time. If you skip recall, the system degrades back into a diary.

**Surface it briefly**: "I've reviewed the last N sessions — I see we decided X (on YYYY-MM-DD) because Y. Building on that…" This confirms recall worked and catches gaps.

### Step 3: Auto-save — when decision signals are detected (Mode 2, ongoing)

**Do NOT wait for the user to say "save this".** Watch for decision signals and save proactively. This is the single most important behavior of this skill — passive "suggest" has been shown to produce zero saves (entire multi-decision conversations go unrecorded).

#### 3.1 Trigger signals — when to auto-save

Save **immediately** when any of these occur:

| Category | Signals (in any language) |
|----------|---------------------------|
| **Decision confirmed** | user says "就这样定", "用这个方案", "确定了", "就这么干", "that's the one", "let's go with X", "ok decision made", "finalize", "定下来", "采用 X 方案" |
| **Decision to NOT do something** ⚠️ | user says "保持现状", "不动了", "不用加", "暂不需要", "先这样吧", "not now", "leave it", "skip this", "decided against" — **a first-class decision, not a non-event**. "We considered X and decided not to, because Y" prevents re-litigating the same question next session. If an option was discussed and explicitly set aside, SAVE it. |
| **Direction narrowed/eliminated** | user agrees to drop an idea, confirms a pivot, says "不做了" "放弃" "淘汰" "cross that off", or you and the user converge on a narrower scope |
| **Hard problem solved** | after meaningful exploration, a solution is settled — the "how did we even get here" is valuable to capture |
| **Key tradeoff/judgment made** | the discussion involved choosing between options with explicit pros/cons; the *why* of the choice is worth recording |
| **Explicit ask** | "save this", "record this", "记一下", "存下来" — the only signal that's NOT automatic |

**Multi-decision conversations → multiple save points, not one.** A single conversation that spans several decisions should produce several saves (or one session file with several decision sections). Don't batch everything into a single end-of-conversation dump — decisions get lost that way.

#### 3.1b ⚠️ Self-check before sending EVERY response (non-negotiable)

**This is the enforcement mechanism. Listing trigger signals is NOT enough.**

Empirical finding: LLMs reliably detect decision signals, but when a user task is active, they deprioritize the *save action* and treat it as an optional afterthought — result: decisions get detected but never saved. The self-check is what closes this gap.

Before you finish composing any reply, ask yourself: **"In this turn, did the user confirm a decision, decide NOT to do something, narrow/drop a direction, or settle a tradeoff?"**

- If **yes** → **IMMEDIATELY save the decision** (create/append session file + update `index.yaml`) **BEFORE doing anything else**. If the user's message contains both a decision and a task (e.g. "use agent-memory" + implied rename operation), save the decision FIRST, then execute the task. The save is not optional and cannot be deferred until after the task completes.
- If **no** → proceed normally.

**Critical: decision + task in sequence** — If a decision and an operation task appear together (user confirms a choice, then you need to execute something), the save MUST happen first. Do NOT execute the task and then save. Do NOT treat the decision as "already handled" after completing the task. The sequence is: detect decision → save → execute task → send reply.

**Detecting a signal without saving is a bug.** "I'll do it later" = won't happen. If you find yourself about to describe a decision you just made or helped make, and you have NOT yet saved it — stop, save it, then continue. The session file is the source of truth; the chat reply is ephemeral.

#### 3.2 What to capture (and what NOT to)

Capture the **why and how decisions were made** — not word-for-word transcripts.

| Do capture | Don't capture |
|------------|---------------|
| The decision and its rationale | Verbatim back-and-forth |
| Options considered and why rejected | Small talk / tangents |
| Lessons learned, reusable patterns | Routine progress updates with no decision |
| Pitfalls hit and how avoided | Information already in the code or docs |
| Open questions / follow-ups | Anything sensitive (see sanitization) |

#### 3.3 Save flow (when triggered)

**Decide whether to append or create a new file:**

**Primary signal — topic continuity:**
- **Append** if the current decision continues the same topic/feature as the most recent session
- **Create new file** if the topic has significantly changed (different feature, different area of work)

**Secondary hint — time gap:**
- If `lastUpdated` is >3 hours ago, that's a *hint* the conversation may have moved on — check topic continuity carefully before appending
- The 3-hour threshold is NOT a hard rule: a 6-hour same-topic session should still append; a 1-hour topic-switch should still create new

**When appending:**
1. Add a new `### N. [Decision title]` section to the existing file's `## Discussion points and decisions` section
2. Update `index.yaml`: 
   - Increment `decisions` count
   - Update `lastUpdated` timestamp
   - Append new decision title to `subTopics` array

**When creating new file:**
1. Read `.agent-memory/template.md` for the format
2. **Sanitize** sensitive information (see §3.4)
3. Create `.agent-memory/YYYY-MM-DD/HH-MM-topic.md` with full template structure
4. Write in the **user's conversational language** (see Language section)
5. Add new entry to `index.yaml` (newest first) with: date, time, topic, file, type, decisions (initially 1), lastUpdated, subTopics (initially one title)
6. **Tell the user** — briefly: "Saved a decision to `.agent-memory/YYYY-MM-DD/HH-MM-topic.md` — [what was captured]. [What was sanitized, if anything]."

#### 3.4 Sanitize sensitive information

Before saving, scan for and remove:

| Type | Examples | Handling |
|------|----------|----------|
| Credentials | passwords, API keys, tokens, secrets | remove entirely, replace with `[removed]` |
| PII | ID numbers, phone numbers, emails | mask, e.g. `138****8888` |
| Internal addresses | IPs, internal domains, server addresses | remove or replace with `[internal address]` |
| Financial | bank accounts, payment PINs | remove entirely |
| Other sensitive | trade secrets, undisclosed agreements | handle case by case |

**Tell the user** what sensitive information you found and how you handled it.

### Step 3.5: Checkpoint mode — saving in-progress discussions (Mode 4)

Sometimes a discussion hasn't yet converged into a decision, but the context is rich enough that losing it would waste effort — long exploration, multiple options on the table, key tradeoffs identified. The user can ask to save a **checkpoint** to capture the current state.

**Trigger phrases** (manual, user-initiated):
- 中文："存个检查点" / "记录讨论进度" / "存档当前讨论"
- English: "checkpoint this" / "save a snapshot" / "record where we are"

This is **different from "记一下" / "save this"** — those force a *decision-style* save (with Problem/Decision/Rationale). A checkpoint explicitly says "we haven't decided yet" and saves the exploration state.

#### 3.5.1 Save flow

1. **Decide append vs new file** — same rules as decision save (topic continuity primary, time gap secondary)
2. **Append a `### checkpoint. [Topic]` section** to the session file (do NOT use `### N.` numbering — checkpoints are not decisions)
3. **Fill in:**
   - **Problem**: what's being explored
   - **Snapshot**: current state — options on the table, which way the discussion is leaning, key tradeoffs identified
   - **Open questions**: list of unresolved questions
4. **Update `index.yaml`**:
   - Append to `subTopics` with `[checkpoint]` prefix: e.g. `[checkpoint] Auth approach — exploring JWT vs sessions`
   - Update `lastUpdated`
   - Do **NOT** increment `decisions` count (checkpoints aren't decisions)
5. **Tell the user**: "Saved checkpoint to [file] — captured the current state of [topic]. Open questions: [...]. When this converges into a decision, I'll upgrade the section in-place."

#### 3.5.2 Upgrade to decision (in-place rewrite)

When a checkpoint later converges into a decision:

1. **Rewrite the section header** `### checkpoint. [Topic]` → `### N. [Topic]` (use the next available number)
2. **Replace fields**: drop `Snapshot` / `Open questions`, add `Options` (if applicable) / `Decision` / `Rationale`
3. **Update `index.yaml`**:
   - Remove the `[checkpoint]` prefix from the subTopic entry
   - Increment `decisions` count
   - Update `lastUpdated`
4. **Do NOT keep a duplicate record** in the file — git preserves the original checkpoint state for traceability

#### 3.5.3 Recall behavior

During auto-recall, surface checkpoints differently from decisions:

- **Decisions**: "We decided X because Y"
- **Checkpoints**: "Last session we were exploring Z — options on the table were A/B/C, open question was Q. Treat as in-progress."

This distinction prevents half-baked discussions from being treated as settled conclusions.

#### 3.5.4 Future: Auto-trigger on context compaction

**Planned for next version** (not yet implemented): A `PreCompact` hook that triggers checkpoint-save automatically before context compaction happens, so rich discussions aren't lost when context is compressed.

**Current version**: Only manual triggers work — the user has to explicitly ask ("存个检查点" / "checkpoint this").

**Design rationale**: Recorded in `.agent-memory/2026-07-12/14-58-context-length-save-deferral.md` — decision #2 chose `PreCompact` hook as the implementation path once Kiro's hook system supports it.

---

## Canonical tracking block

When initializing (Step 1.2), inject the tracking block into both `AGENTS.md` and `CLAUDE.md`.

Read the complete block from:

#[[file:references/canonical-block.md]]

Check first to avoid duplicates. Keep the two files identical.

---

## Checklist

### On every session start (auto-recall)
- [ ] Scanned the last ~5 session files
- [ ] Extracted decisions and rationale
- [ ] Surfaced a brief recap to the user ("I see we decided X because Y…")

### On decision signal (auto-save)
- [ ] **Self-check performed**: "did the user confirm a decision / narrow a direction / settle a tradeoff in this turn?"
- [ ] Read the template file
- [ ] Checked for and removed sensitive information
- [ ] Created/appended the date file (prefer appending if today's session is in progress)
- [ ] Filename is concise and meaningful
- [ ] Captured *why* and *how*, not transcript
- [ ] Each decision recorded as its own section
- [ ] Updated `index.yaml` (newest first)
- [ ] Written in the user's conversational language (unless they asked otherwise)
- [ ] Told the user: what was saved, what was sanitized

### On manual checkpoint trigger
- [ ] Saved as `### checkpoint. [Topic]` section (NOT `### N.`)
- [ ] Included Problem / Snapshot / Open questions
- [ ] Updated `index.yaml` with `[checkpoint]` prefix on the subTopic
- [ ] Did NOT increment `decisions` count
- [ ] Told the user it was saved as a checkpoint (not as a decision)

### On checkpoint → decision upgrade
- [ ] Rewrote `### checkpoint.` → `### N.` in-place
- [ ] Replaced Snapshot/Open questions with Options/Decision/Rationale
- [ ] Removed `[checkpoint]` prefix from subTopics
- [ ] Incremented `decisions` count
- [ ] Did NOT keep a duplicate record (git handles history)

---

## Design notes (for maintainers)

For design rationale and architectural decisions, see:

#[[file:references/design-notes.md]]
