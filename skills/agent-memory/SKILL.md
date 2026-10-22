---
name: agent-memory
description: Auto-saves decisions during conversations and auto-recalls them at session start across agents and machines. Use when user mentions memory setup, saving conversations, checkpoints, or asks for help.
keywords: memory, session, tracking, decisions, recall, knowledge, conversation, notes, remember, 记录, 记忆
---

# Agent Memory

Save decisions as they happen and recall them across sessions and agents using `.agent-memory/`.

## Request routing

| Request | Action |
|---------|--------|
| "设置记忆", "set up / initialize / enable memory" | Follow Initialize below. |
| "记忆帮助", "memory help", questions about usage or save triggers | Read `references/help-text.md`, show it in the user's language, and emphasize affirmative decision language. |
| "存个检查点", "记录讨论进度", "存档当前讨论", "checkpoint this", "save a snapshot", "record where we are" | Save an unfinished discussion as a checkpoint using Saving records. Checkpoints are manual-only in this version. |
| "记一下", "存下来", "save this", "record this" | Save a decision using Saving records. |
| "漏了", "这条刚才没记下来", "you didn't save that" | Recover the missed decision, then append a gap entry to `.agent-memory/IMPROVEMENTS.md` using its entry template. |

For help-only requests, show help. For mixed requests, also perform the requested operations; a usage question alone does not request a save or checkpoint. Confirmed decisions follow the save-first rule below before other operations or help.

Once initialized, recall and decision auto-save apply throughout normal work without explicit requests.

## Initialize

1. If an old `agent-memory/` directory exists, read and follow the migration guide, including confirmation before moving data:
   #[[file:references/migration.md]]
2. Read and follow the initialization guide:
   #[[file:references/initialization.md]]

Completion: scaffold files exist, existing memory is preserved, and both `AGENTS.md` and `CLAUDE.md` contain exactly one current tracking block. Keep those blocks identical and preserve all content outside them; the guide defines the replacement boundaries.

## Recall

At session start, or when the user references prior work or decisions, recall before responding substantively:

1. Read `.agent-memory/index.yaml` and scan **all** sessions' `subTopics` for relevance. If `sessions` is empty, report that no memories have been saved and continue the user's task.
2. Open matching session files. Only when no topics match, read the most recent ~5 sessions in full.
3. Within `## Discussion points and decisions`, extract by heading prefix:

   | Prefix | Fields | Meaning |
   |--------|--------|---------|
   | `### N.` | Problem / Decision / Rationale | Settled decision |
   | `### checkpoint.` | Problem / Snapshot / Open questions | Unfinished discussion |

   Use the same distinction for fallback files. Also extract reusable lessons from `## Lessons learned`; other sections have lower recall priority.
4. Briefly surface relevant conclusions and their rationale, identifying checkpoints as unfinished. Carry settled decisions forward and respect prior rejections.

## Decision auto-save

Save immediately when any signal occurs, without waiting for a save request:

| Signal | Examples |
|--------|----------|
| Decision confirmed | "就这样定", "用这个方案", "确定了", "就这么干", "采用 X 方案", "that's the one", "let's go with X", "finalize" |
| Decision against an option | "保持现状", "不动了", "不用加", "暂不需要", "先这样吧", "not now", "leave it", "skip this", "decided against" |
| Direction narrowed | User drops an idea, confirms a pivot, or agrees on a narrower scope. |
| Hard problem solved | Exploration settles on a solution worth retaining. |
| Tradeoff settled | A choice is made between options; capture why. |
| Explicit save | "记一下", "存下来", "save this", "record this" |

Decisions against doing something count as decisions. Save each decision when it happens; give multiple decisions separate sections instead of deferring them all to the conversation's end.

### Required self-check before every response

Ask: **Did this turn confirm a decision, reject or narrow a direction, settle a hard problem or tradeoff, or explicitly request a save?**

If yes, **save the record and update `index.yaml` before executing other tasks or replying**. For a decision plus an operation request, the order is **detect → save → execute → reply**. Detecting a signal without saving is a bug. If no signal occurred, continue normally.

## Saving records

Apply these shared steps to decisions, checkpoints, and checkpoint upgrades:

1. Read `.agent-memory/template.md`, the index, and any target session file. Capture the problem, choice or exploration state, rationale, options, lessons, and follow-ups as appropriate; omit transcripts, small talk, routine status, and facts already recorded in code or docs.
2. Sanitize content using Sanitization below. Write titles and body in the user's language unless another language was explicitly requested. Preserve the template's English structural headings and field names, plus `### N.` and `### checkpoint.` prefixes.
3. Choose the target:
   - If a decision settles an existing checkpoint, upgrade that section in place.
   - Otherwise, append when the topic continues the most recent session; create a new file when the topic changes or no session exists.
   - A gap over 3 hours is a hint to recheck topic continuity, not a cutoff: a 6-hour continuation can append; a 1-hour topic switch creates new.
4. Write inside `## Discussion points and decisions` using the matching operation:

   | Operation | Session content | Index update |
   |-----------|-----------------|--------------|
   | Append decision | Add `### N. [Topic]` with Problem / Decision / Rationale; Options optional. | Add the title to `subTopics`; increment `decisions`. |
   | New decision session | Create a full-template file with the decision section. | Create a full index entry with `decisions: 1` and the title in `subTopics`. |
   | Append checkpoint | Add `### checkpoint. [Topic]` with Problem / Snapshot / Open questions. | Add `[checkpoint] [Topic]` to `subTopics`; leave `decisions` unchanged. |
   | New checkpoint session | Create a full-template file with the checkpoint section and no decision placeholders. This includes the first saved record. | Create a full index entry with `decisions: 0` and one `[checkpoint] [Topic]` subTopic. |
   | Upgrade checkpoint | Rewrite the same section as `### N. [Topic]`, using the next available decision number; replace Snapshot / Open questions with Decision / Rationale and optional Options. | Remove the corresponding `[checkpoint]` prefix; increment `decisions`. Keep one record; git retains prior states. |

5. For new sessions, use `.agent-memory/YYYY-MM-DD/HH-MM-topic.md` and add the index entry newest first with date, time, topic, file, type, decisions, lastUpdated, and subTopics. Update `lastUpdated` for every save or upgrade.
6. Before replying, verify the saved section and index agree: the file exists, `subTopics` reflects its decision and checkpoint titles, and `decisions` counts only decisions. Briefly tell the user where the record was saved, what was captured, and what was sanitized, if anything. For checkpoints, identify the open questions and unfinished status.

## Sanitization

Before any save, remove or mask sensitive content:

| Content | Handling |
|---------|----------|
| Passwords, API keys, tokens, secrets | Remove; use `[removed]`. |
| ID numbers, phone numbers, emails | Mask, e.g. `138****8888`. |
| Internal IPs, domains, server addresses | Remove or use `[internal address]`. |
| Bank accounts, payment PINs | Remove entirely. |
| Trade secrets, undisclosed agreements | Sanitize case by case. |

## Design notes

For implementation history, rationale, and deferred work, read:
#[[file:references/design-notes.md]]
