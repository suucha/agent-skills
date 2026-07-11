# Agent Instructions

This project develops AI agent skills, specifically the `agent-memory` skill.

## Development Workflow — Skill File Synchronization

When developing the `agent-memory` skill, files in `skills/agent-memory/` serve as the **single source of truth**.

### After modifying any file in `skills/agent-memory/`:

**ALWAYS synchronize changes to all skill directories.** Use **delete-then-copy** — `Copy-Item -Recurse -Force` and `cp -r` both silently copy *into* an existing destination instead of replacing it, creating a nested `agent-memory/agent-memory/`. After the first sync the destination always exists, so every subsequent sync hits this trap unless you remove first.

**PowerShell** (run in PowerShell window, or `powershell.exe -Command` from another shell):

```powershell
Remove-Item -Recurse -Force ".kiro\skills\agent-memory", ".agents\skills\agent-memory", ".claude\skills\agent-memory"
Copy-Item -Recurse "skills\agent-memory" ".kiro\skills\agent-memory"
Copy-Item -Recurse "skills\agent-memory" ".agents\skills\agent-memory"
Copy-Item -Recurse "skills\agent-memory" ".claude\skills\agent-memory"
```

**Or as a one-liner:**

```powershell
Remove-Item -Recurse -Force ".kiro\skills\agent-memory", ".agents\skills\agent-memory", ".claude\skills\agent-memory"; Copy-Item -Recurse "skills\agent-memory" ".kiro\skills\agent-memory"; Copy-Item -Recurse "skills\agent-memory" ".agents\skills\agent-memory"; Copy-Item -Recurse "skills\agent-memory" ".claude\skills\agent-memory"
```

**Bash** (Git Bash, WSL, or Claude Code's shell on Windows):

```bash
rm -rf .kiro/skills/agent-memory .agents/skills/agent-memory .claude/skills/agent-memory && \
cp -r skills/agent-memory .kiro/skills/agent-memory && \
cp -r skills/agent-memory .agents/skills/agent-memory && \
cp -r skills/agent-memory .claude/skills/agent-memory
```

### Why this is necessary:

- **Junction/symlinks don't work** with Kiro/Claude/opencode skill loading
- **Real file copies are required** for each tool to recognize the skill
- **Three copies** ensure the skill works across all agent platforms
- **Manual sync** is needed because file watchers don't trigger on source changes
- **Delete-then-copy** is required (not plain `Copy-Item -Recurse` / `cp -r`) — see the note above about nested-directory trap

### When to sync:

- ✅ After editing `SKILL.md`
- ✅ After editing `README.md`
- ✅ After adding/modifying files in `references/`
- ✅ After adding/modifying files in `evals/`
- ✅ Before testing the skill in Kiro/Claude/opencode
- ✅ Before committing changes to git

### Automation option:

If you find yourself syncing frequently, you can create a PowerShell script or git hook to automate this. For now, manual sync ensures you're aware of when changes propagate.

---

## Self-Hosting Validation

This project uses the `agent-memory` skill to validate itself during development (dogfooding).

Once the skill is loaded (after copying to `.kiro/skills/`), it will:
- Auto-save decisions to `agent-memory/` at the project root
- Auto-recall prior decisions at session start
- Record missed saves to `agent-memory/IMPROVEMENTS.md`

This creates a feedback loop where the skill validates itself in real-world usage.

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

Before responding substantively, read `agent-memory/index.yaml` for the recent session list (newest first, up to ~5). For each session:
- Read `subTopics` from index.yaml (quick overview of what was discussed)
- Open the file and extract all `### [title]` sections from `## Discussion points and decisions`
- For each section, extract: **Problem**, **Decision**, **Rationale**
- Skip low-priority sections: Execution plan, Related content, Follow-ups

Carry that context forward and surface it briefly: "I've reviewed the last N sessions — we decided X because Y…" This ensures you never re-litigate settled decisions or propose approaches already rejected. **This is what makes the system "memory" rather than an archive.**

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
   - Append if: session file exists for today AND `lastUpdated` in index.yaml is within ~3 hours AND topic is related
   - Create new if: no session file for today OR last update >3 hours ago OR topic changed significantly
4. **When appending**:
   - Add new `### N. [Decision title]` section to `## Discussion points and decisions`
   - Update `index.yaml`: increment `decisions`, update `lastUpdated`, append to `subTopics`
5. **When creating new**:
   - Create `agent-memory/YYYY-MM-DD/HH-MM-topic.md` with full template structure
   - Add entry to `index.yaml` (newest first) with: date, time, topic, file, type, decisions (initially 1), lastUpdated, subTopics (initially one title)
6. **Fill it in using the template** — capture the *why* and *how*, not transcripts
7. **Tell the user** — where it was saved, what was captured, what was sanitized

**index.yaml structure**:
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
      - "Decision 2 title"
```

- `subTopics`: List of all `###` headings from `## Discussion points and decisions` (auto-extracted)
- `lastUpdated`: Timestamp of last modification (used to decide append vs create new)
- `decisions`: Count of decision points in the file

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
