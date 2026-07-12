---
name: agent-memory
description: Set up an AI memory system that automatically captures important decisions during conversations and recalls them across agents and machines. Unlike a passive archive, this skill AUTO-SAVES when decision signals are detected (no need for the user to ask) and AUTO-RECALLS prior decisions at the start of each session. You MUST use this skill whenever the user mentions set up memory, initialize memory, enable memory, conversation memory, saving a conversation, session notes, AI notes, knowledge retention, team collaboration, or wants the AI to remember previous discussions — e.g. "set up memory", "initialize memory", "enable memory", "save this conversation", "record this", "session notes", "remember what we decided", "设置记忆", "初始化记忆", "启用记忆", or "记一下". Also use this skill when the user wants to save an in-progress discussion snapshot (not yet a decision) — e.g. "存个检查点", "记录讨论进度", "checkpoint this", "save a snapshot". Also applies when initializing a new project and establishing memory conventions. ALSO use this skill when the user asks what the skill does or how to use it — e.g. "memory help", "记忆帮助", "how does memory work", "怎么用记忆", "what did you save", "what triggers a save" — display the on-demand help text described in the "On-demand help" section.
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

Show this when Mode 3 is triggered. Adapt the language to match the user's (Chinese ↔ English); keep the structure.

---

**What this skill does**: Captures decisions during our conversations and recalls them next session, so any agent picking up this project knows what was already decided and why — you don't re-litigate settled questions.

**You don't need to do anything special.** Auto-save and auto-recall run on their own. But one habit dramatically improves how well it works:

### Use affirmative language when you've decided ⚠️

This skill detects decisions from your language. **Ambiguous phrasing gets missed; strong signals get saved.**

✅ **Strong signals** (reliably saved):
- 中文：「就选 A」「用 A 方案」「确定了」「就这样定」「就这么干」
- English: "let's go with X", "that's the one", "finalize X", "decision made"

❌ **Weak signals** (often missed):
- 中文：「选 A 吧」「可能用 A」「先试试 A?」「感觉 A 好一点」「先用 A 看看」
- English: "maybe A", "I guess A", "let's try A?", "I think A"

**If you've made up your mind, say so plainly.** If you're still exploring, that's fine — just don't expect a save until you commit.

### Other things worth knowing

- **Decisions to NOT do something count too.** "不做 X，因为 Y" / "we're not doing X because Y" is a first-class decision — recording it prevents the same question coming back next session.
- **Manual triggers**:
  - "记一下" / "save this" / "record this" — force a save right now
  - "存个检查点" / "checkpoint this" / "记录讨论进度" — save an in-progress discussion snapshot (not yet a decision); recorded as `### checkpoint.` section, surfaced differently during recall
  - "漏了" / "这条刚才没记下来" / "you didn't save that" — flag a missed save (the system self-improves via `IMPROVEMENTS.md`)
  - "记忆帮助" / "memory help" — show this help again
- **Where things live**:
  - `agent-memory/index.yaml` — session index (newest first)
  - `agent-memory/YYYY-MM-DD/HH-MM-topic.md` — individual session files
  - `agent-memory/IMPROVEMENTS.md` — backlog of missed saves (the system's own bug tracker)

---

## Core workflow

### Step 1: Initialize (Mode 1)

#### 1.1 Create the directory structure

At the project root, create:

```
agent-memory/
├── README.md                    # usage instructions
├── IMPROVEMENTS.md              # rule-gap log (self-maintaining backlog of missed saves)
├── template.md                  # session template (copied from this skill)
├── .gitignore                   # exclude sensitive info (optional)
└── YYYY-MM-DD/                  # organized by date
    ├── HH-MM-topic.md
    └── ...
```

Copy these files from the skill's `references/` directory:
- `references/README.md` → `agent-memory/README.md`
- `references/IMPROVEMENTS.md` → `agent-memory/IMPROVEMENTS.md`
- `references/template.md` → `agent-memory/template.md`

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
1. Read `agent-memory/index.yaml` → scan ALL sessions' `subTopics` arrays
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
1. Read `agent-memory/template.md` for the format
2. **Sanitize** sensitive information (see §3.4)
3. Create `agent-memory/YYYY-MM-DD/HH-MM-topic.md` with full template structure
4. Write in the **user's conversational language** (see Language section)
5. Add new entry to `index.yaml` (newest first) with: date, time, topic, file, type, decisions (initially 1), lastUpdated, subTopics (initially one title)
6. **Tell the user** — briefly: "Saved a decision to `agent-memory/YYYY-MM-DD/HH-MM-topic.md` — [what was captured]. [What was sanitized, if anything]."

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

#### 3.5.4 Auto-trigger (deferred to next version)

A `PreCompact` hook will trigger checkpoint-save automatically before context compaction happens. For now, only manual triggers work — the user has to ask.

---

## Canonical tracking block

This is the block to inject into both `AGENTS.md` and `CLAUDE.md` during initialization. Keep the two files identical.

````markdown

## AI Session Tracking

### Directory Structure

```
agent-memory/
├── index.yaml            # session index (auto-maintained, newest first)
├── IMPROVEMENTS.md       # rule-gap log (self-maintaining backlog of missed saves)
├── template.md           # session file template
└── YYYY-MM-DD/
    └── HH-MM-topic.md
```

### Self-improvement loop — handling missed saves

This system has a blind-spot: it relies on trigger signals, and those signals
are incomplete by nature. When a decision was discussed but NOT auto-saved,
the system has no way to know — unless the user points it out.

**When the user says "这条刚才没记下来" / "you didn't save that" / "that should have been saved" / "漏了":**

1. **Save the decision** that was missed (append to today's session file + update index.yaml) — this is recovery.
2. **Append a gap entry to `agent-memory/IMPROVEMENTS.md`** — this is prevention. Use the entry template at the bottom of that file. Record: date, symptom (what was discussed but not saved), root cause (which signal was missing/ambiguous), status (open).

This makes the system its own bug tracker: gaps accumulate in a file (not in
someone's memory), and any future session that wants to improve the rules reads
`IMPROVEMENTS.md` as its work list. **The user never has to route feedback to a
specific "maintainer session" — they flag it wherever they are, and the repo
carries it forward.**

### Auto-recall (at the start of every session)

Two-layer recall — cheap scan first, deep read only when relevant:

1. **Lightweight scan**: read `agent-memory/index.yaml`, scan ALL sessions' `subTopics`, judge which sessions are relevant to the current question
2. **Deep extraction**: for each relevant session, open the file, extract Problem/Decision/Rationale from each `###` subsection under `## Discussion points and decisions`
3. **Fallback**: if no sessions match by subTopics, read the ~5 most recent sessions in full

Surface it briefly: "I've reviewed the last N sessions — we decided X because Y…" This ensures you never re-litigate settled decisions or propose approaches already rejected. **This is what makes the system "memory" rather than an archive.**

### Auto-save (when decision signals are detected)

Save proactively — **do NOT wait for the user to ask.** Trigger immediately when any of these occur:

- **Decision confirmed** — "就这样定", "用这个方案", "确定了", "let's go with X", "finalize"
- **Decision to NOT do something** ⚠️ — "保持现状", "不动了", "不用加", "暂不需要", "先这样吧", "not now", "leave it", "skip this", "decided against". **This is a first-class decision, not a non-event.** "We considered X and decided not to do it, because Y" is exactly the kind of record that prevents re-litigating the same question next session. If an option was discussed and explicitly set aside, SAVE it.
- **Direction narrowed/eliminated** — user agrees to drop an idea, confirms a pivot, or scope converges
- **Hard problem solved** — after exploration, a solution is settled (the "how we got here" is worth capturing)
- **Key tradeoff/judgment made** — discussion involved choosing between options; the *why* is worth recording
- **Explicit ask** — "save this", "record this", "记一下" (the only non-automatic signal)

Multi-decision conversations → multiple save points. Don't batch everything into one end-of-conversation dump.

### ⚠️ Self-check before sending EVERY response (non-negotiable)

Before you finish composing any reply, ask yourself: **"In this turn, did the user confirm a decision, decide NOT to do something, narrow/drop a direction, or settle a tradeoff?"**

- If **yes** → **IMMEDIATELY save the decision** (create/append the session file + update index.yaml) **BEFORE doing anything else**. If the user's message contains both a decision and a task (e.g. "use agent-memory" + implied rename operation), save the decision FIRST, then execute the task. The save is not optional and cannot be deferred until after the task completes.
- If **no** → proceed normally.

**This checkpoint is the enforcement mechanism.** Listing trigger signals is not enough — LLMs deprioritize meta-instructions when a user task is active. The self-check is what forces the save to actually happen. **Detecting a signal without saving is a bug.** "I'll do it later" = won't happen.

**Critical: decision + task in sequence** — If a decision and an operation task appear together (user confirms a choice, then you need to execute something), the save MUST happen first. Do NOT execute the task and then save. Do NOT treat the decision as "already handled" after completing the task. The sequence is: detect decision → save → execute task → send reply.

If you find yourself about to describe a decision you just made or helped make, and you have NOT yet saved it — stop, save it, then continue. The session file is the source of truth; the chat reply is ephemeral.

### Saving process

1. **Read the template** — read `agent-memory/template.md` first to understand the format
2. **Sanitize content** — remove sensitive information (passwords, keys, PII, etc.)
3. **Decide: append or create new**:
   - **Primary signal — topic continuity**: append if the current decision continues the same topic as the most recent session; create new if the topic has significantly changed
   - **Secondary hint — time gap**: if `lastUpdated` is >3 hours ago, check topic continuity carefully (but 3h is NOT a hard rule — a 6h same-topic session still appends; a 1h topic-switch still creates new)
4. **When appending**:
   - Add new `### N. [Decision title]` section to `## Discussion points and decisions`
   - Update `index.yaml`: increment `decisions`, update `lastUpdated`, append to `subTopics`
5. **When creating new**:
   - Create `agent-memory/YYYY-MM-DD/HH-MM-topic.md` with full template structure
   - Add entry to `index.yaml` (newest first) with: date, time, topic, file, type, decisions (initially 1), lastUpdated, subTopics (initially one title)
6. **Fill it in using the template** — capture the *why* and *how*, not transcripts
7. **Tell the user** — where it was saved, what was captured, what was sanitized

### Checkpoints (in-progress discussion snapshots)

Sometimes a discussion hasn't converged into a decision yet, but the context is rich enough to be worth saving. The user can trigger a checkpoint manually:

- 中文："存个检查点" / "记录讨论进度"
- English: "checkpoint this" / "save a snapshot"

**Save flow:**

1. **Decide append vs new file** — same rules as decision save (topic continuity primary, time gap secondary)
2. Append a `### checkpoint. [Topic]` section to the session file with `Problem` / `Snapshot` / `Open questions` fields
3. Update `index.yaml`: append `[checkpoint] [Topic]` to `subTopics` (with prefix), update `lastUpdated`, do **NOT** increment `decisions`
4. Tell the user it was saved as a checkpoint

**Upgrade to decision (in-place rewrite):**

When the checkpoint later converges into a decision:

1. Rewrite `### checkpoint. [Topic]` → `### N. [Topic]`
2. Replace `Snapshot` / `Open questions` with `Options` / `Decision` / `Rationale`
3. Update `index.yaml`: remove `[checkpoint]` prefix, increment `decisions`, update `lastUpdated`

Git preserves the original checkpoint for traceability — no need to keep a duplicate record in the file.

### index.yaml structure

```yaml
sessions:
  - date: "YYYY-MM-DD"
    time: "HH:MM"
    topic: "Session topic"
    file: "YYYY-MM-DD/HH-MM-topic.md"
    type: "coding / design / discussion / etc"
    decisions: 3
    lastUpdated: "YYYY-MM-DD HH:MM"
    subTopics:
      - "Decision 1 title"
      - "[checkpoint] In-progress topic title"   # prefix marks checkpoints
      - "Decision 2 title"
```

- **subTopics**: List of all `###` headings from `## Discussion points and decisions` section (auto-extracted when saving); entries prefixed with `[checkpoint]` are in-progress snapshots, not decisions
- **lastUpdated**: Timestamp of last modification (used to decide append vs create new)
- **decisions**: Count of decision points in the file (does NOT include checkpoints)

### Language

Write each saved session in the **same language the user is conversing in** — unless the user explicitly asks for a specific language.

**Default behavior**: Match the user's language
- If the user writes in Chinese, the whole session file (section headers and content) is in Chinese
- If in English, English; and so on for any other language
- Do not translate the user's words into another language

**When user specifies a different language**:
- User: "save this in English" / "用英文记录" → save in English regardless of conversation language
- Common in cross-border teams: conversation in Chinese, but records in English for team sharing
- Honor the user's choice exactly as specified

The record should read naturally for the team that will use it.

### Notes

- Focus on the "why" and "how decisions were made", not word-for-word transcripts
- Record each decision point separately so they're easy to look up later
- Keep filenames concise but meaningful
- Sensitive information must be sanitized before saving
````

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

- **Why auto-save over "suggest"**: the previous "proactively suggest" wording left the choice to the agent, which defaults to not-suggesting. Empirically, multi-decision conversations produced zero saves until the user manually intervened. Auto-save fixes the root cause.
- **Why the self-check checkpoint (added after testing)**: in live testing, auto-recall worked (it runs at session start, when no user task competes for attention) but auto-save failed (it runs mid-conversation, and the active user task causes the LLM to deprioritize the meta-instruction). The signal table tells the agent *what* to detect, but the self-check — a forced checkpoint at the end of every reply — is what forces the save to actually *happen*. Listing triggers ≠ executing saves.
- **Why auto-recall is non-negotiable**: a memory that isn't read is just a diary. The recall step is what makes this valuable across agents and machines — the core use case ("switch machines, new agent picks up where the last left off") only works if recall runs.
- **Why both `AGENTS.md` and `CLAUDE.md`**: each tool reads only its own config file. If you update one, users of the other tool never see the rules.
- **Why `index.yaml`**: scanning a directory of markdown files every session start is wasteful and slow. A single index file lets auto-recall read one file to find the recent sessions, then pull only those. The index is auto-maintained — every save appends an entry.
- **Cross-agent memory**: the `agent-memory/` directory (plain markdown + yaml, committed to git) is the **provider-neutral memory layer**. Each agent vendor locks memory inside its own format (Claude's, Cursor's, etc.); this directory is the escape hatch. The auto-recall step is what bridges vendor-specific sessions into this shared layer.
