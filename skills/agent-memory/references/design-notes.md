# Design Notes (for maintainers)

- **Why auto-save over "suggest"**: the previous "proactively suggest" wording left the choice to the agent, which defaults to not-suggesting. Empirically, multi-decision conversations produced zero saves until the user manually intervened. Auto-save fixes the root cause.
- **Why the self-check checkpoint (added after testing)**: in live testing, auto-recall worked (it runs at session start, when no user task competes for attention) but auto-save failed (it runs mid-conversation, and the active user task causes the LLM to deprioritize the meta-instruction). The signal table tells the agent *what* to detect, but the self-check — a forced checkpoint at the end of every reply — is what forces the save to actually *happen*. Listing triggers ≠ executing saves.
- **Why auto-recall is non-negotiable**: a memory that isn't read is just a diary. The recall step is what makes this valuable across agents and machines — the core use case ("switch machines, new agent picks up where the last left off") only works if recall runs.
- **Why both `AGENTS.md` and `CLAUDE.md`**: each tool reads only its own config file. If you update one, users of the other tool never see the rules.
- **Why `index.yaml`**: scanning a directory of markdown files every session start is wasteful and slow. A single index file lets auto-recall read one file to find the recent sessions, then pull only those. The index is auto-maintained — every save appends an entry.
- **Cross-agent memory**: the `.agent-memory/` directory (plain markdown + yaml, committed to git) is the **provider-neutral memory layer**. Each agent vendor locks memory inside its own format (Claude's, Cursor's, etc.); this directory is the escape hatch. The auto-recall step is what bridges vendor-specific sessions into this shared layer.

## Deferred checkpoint automation

This version supports manual checkpoints only. A `PreCompact` hook to save an unfinished discussion before context compaction is planned for a future version; it is not implemented. The design keeps checkpoints separate from settled decisions and upgrades them in place when the discussion converges.

## Main-file organization

The main skill keeps request routing, recall, the mandatory save-first self-check, and a shared save flow together. Initialization and help are reached only for their respective requests. Repeated explanations and future plans live here so everyday execution reads the operational rules once.
