---
name: agent-memory
description: Set up an AI memory system that automatically captures important decisions during conversations and recalls them across agents and machines. Unlike a passive archive, this skill AUTO-SAVES when decision signals are detected (no need for the user to ask) and AUTO-RECALLS prior decisions at the start of each session. You MUST use this skill whenever the user mentions setting up session tracking, memory, conversation memory, saving a conversation, session notes, AI notes, knowledge retention, team collaboration, or wants the AI to remember previous discussions — e.g. "set up session tracking", "save this conversation", "record this", "session notes", "remember what we decided", or "记一下". Also applies when initializing a new project and establishing session-tracking conventions.
---

# Agent Memory

A memory system that turns conversations into **reusable knowledge**: it auto-saves decisions when they happen, and auto-recalls them the next time any agent starts work on the project.

The two mechanisms that make this "memory" and not just an "archive":

1. **Auto-save** — detect decision signals and save *without waiting for the user to ask*
2. **Auto-recall** — read prior decisions at the start of a session and surface them, so no agent ever starts blind

## Why both matter

Saving alone = a diary no one reads. Recalling alone = nothing to recall. The value is the loop: **capture → store → recall → build on prior decisions**. A new agent (or the same agent on a new machine) should be able to pick up where the last one left off.

---

## Two usage modes

### Mode 1: Initialize the memory system

When the user says "set up session tracking", "configure memory", or something similar, run the initialization.

**Always create/update both files** (regardless of whether they already exist):
1. **AGENTS.md** (read by opencode)
2. **CLAUDE.md** (read by Claude Code)

Updating both matters because each tool reads only its own config file — if you touch just one, users of the other tool never see the rules.

### Mode 2: Auto-save & auto-recall (ongoing, during every session)

This is the default behavior once initialized — it runs continuously, not on demand.

---

## Core workflow

### Step 1: Initialize (Mode 1)

#### 1.1 Create the directory structure

At the project root, create:

```
ai-sessions/
├── README.md                    # usage instructions
├── IMPROVEMENTS.md              # rule-gap log (self-maintaining backlog of missed saves)
├── template.md                  # session template (copied from this skill)
├── .gitignore                   # exclude sensitive info (optional)
└── YYYY-MM-DD/                  # organized by date
    ├── HH-MM-topic.md
    └── ...
```

Also create `IMPROVEMENTS.md` with an Open / Resolved section and an entry template at the bottom (see `references/IMPROVEMENTS.md` for the seed format).

Copy `references/template.md` from this skill into `ai-sessions/template.md`.

#### 1.2 Update AGENTS.md and CLAUDE.md

Append the tracking block to **both** files (see the canonical block in `## Canonical tracking block` below). Check first to avoid duplicates.

### Step 2: Auto-recall — at the start of a session (Mode 2, ongoing)

**When the conversation begins, or when the user references prior work / a prior decision, ALWAYS do this before responding substantively:**

1. Read `ai-sessions/template.md` to understand the format (so you can parse stored files)
2. Scan `ai-sessions/` for recent session files (newest first, last ~5 files)
3. Extract the **decisions and rationale** from each (the `## Discussion points and decisions` and `## Lessons learned` sections are the highest-value parts)
4. Carry that context into the current conversation — do not re-litigate settled decisions, do not propose approaches already rejected

**This is what makes it memory.** The point is: when the user opens a new session or switches machines, the agent already knows what was decided last time. If you skip recall, the system degrades back into a diary.

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

1. Read `ai-sessions/template.md` for the format
2. **Sanitize** sensitive information (see §3.4)
3. Decide: append a decision section to the current day's session file, or create a new one — prefer appending to the existing day file if a session is already in progress
4. Write in the **user's conversational language** (whole file — see `references/template.md` language note)
5. **Tell the user** — briefly: "Saved a decision to `ai-sessions/YYYY-MM-DD/HH-MM-topic.md` — [what was captured]. [What was sanitized, if anything]."

The tell-the-user step matters: it makes auto-save visible (so the user can correct course if it saved something trivial) and builds trust in the system.

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

---

## Canonical tracking block

This is the block to inject into both `AGENTS.md` and `CLAUDE.md` during initialization. Keep the two files identical.

````markdown

## AI Session Tracking

### Directory Structure

```
ai-sessions/
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
2. **Append a gap entry to `ai-sessions/IMPROVEMENTS.md`** — this is prevention. Use the entry template at the bottom of that file. Record: date, symptom (what was discussed but not saved), root cause (which signal was missing/ambiguous), status (open).

This makes the system its own bug tracker: gaps accumulate in a file (not in
someone's memory), and any future session that wants to improve the rules reads
`IMPROVEMENTS.md` as its work list. **The user never has to route feedback to a
specific "maintainer session" — they flag it wherever they are, and the repo
carries it forward.**

### Auto-recall (at the start of every session)

Before responding substantively, read `ai-sessions/index.yaml` for the recent session list (newest first, up to ~5), then read each referenced file to extract decisions and rationale, and carry that context forward. Surface it briefly: "I've reviewed the last N sessions — we decided X because Y…" This ensures you never re-litigate settled decisions or propose approaches already rejected. **This is what makes the system "memory" rather than an archive.**

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

1. **Read the template** — read `ai-sessions/template.md` first to understand the format
2. **Sanitize content** — remove sensitive information (passwords, keys, PII, etc.)
3. **Create/append the file** — `ai-sessions/YYYY-MM-DD/HH-MM-topic.md` (prefer appending a decision section to today's file if one is in progress)
4. **Fill it in using the template** — capture the *why* and *how*, not transcripts
5. **Update the index** — append an entry to `ai-sessions/index.yaml` (newest first) with date, time, topic, file path, type, and decision count
6. **Tell the user** — where it was saved, what was captured, what was sanitized

### Language

Write each saved session in the **same language the user is conversing in** — unless the user explicitly asks for a specific language (e.g. "save this in English", "用英文记录"). In that case, honor their choice. Otherwise: if the user writes in Chinese, the whole session file (section headers and content) is in Chinese; if in English, English; and so on for any other language. Do not translate the user's words into another language — the record should read naturally for the team that will use it.

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

---

## Design notes (for maintainers)

- **Why auto-save over "suggest"**: the previous "proactively suggest" wording left the choice to the agent, which defaults to not-suggesting. Empirically, multi-decision conversations produced zero saves until the user manually intervened. Auto-save fixes the root cause.
- **Why the self-check checkpoint (added after testing)**: in live testing, auto-recall worked (it runs at session start, when no user task competes for attention) but auto-save failed (it runs mid-conversation, and the active user task causes the LLM to deprioritize the meta-instruction). The signal table tells the agent *what* to detect, but the self-check — a forced checkpoint at the end of every reply — is what forces the save to actually *happen*. Listing triggers ≠ executing saves.
- **Why auto-recall is non-negotiable**: a memory that isn't read is just a diary. The recall step is what makes this valuable across agents and machines — the core use case ("switch machines, new agent picks up where the last left off") only works if recall runs.
- **Why both `AGENTS.md` and `CLAUDE.md`**: each tool reads only its own config file. If you update one, users of the other tool never see the rules.
- **Why `index.yaml`**: scanning a directory of markdown files every session start is wasteful and slow. A single index file lets auto-recall read one file to find the recent sessions, then pull only those. The index is auto-maintained — every save appends an entry.
- **Cross-agent memory**: the `ai-sessions/` directory (plain markdown + yaml, committed to git) is the **provider-neutral memory layer**. Each agent vendor locks memory inside its own format (Claude's, Cursor's, etc.); this directory is the escape hatch. The auto-recall step is what bridges vendor-specific sessions into this shared layer.
