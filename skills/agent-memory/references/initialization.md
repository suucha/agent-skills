# Initialization Guide

This guide walks through the complete initialization process for the agent-memory system.

## Step 1: Create the directory structure

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

## Step 2: Update AGENTS.md and CLAUDE.md

Append the tracking block to **both** files. Check first to avoid duplicates.

Read the complete block from:
#[[file:canonical-block.md]]

**Important**: Always update both files (regardless of whether they already exist). Each tool reads only its own config file — if you touch just one, users of the other tool never see the rules.

---

## Session File Structure (strict format)

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

---

## Language

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
