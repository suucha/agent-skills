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

(gap entries will be added here as they are discovered)

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
