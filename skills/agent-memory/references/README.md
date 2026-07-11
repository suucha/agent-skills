# Agent Memory Sessions

This directory stores the AI agent's memory - decision records that persist across sessions and machines.

## Directory Structure

```
agent-memory/
├── index.yaml              # session index (auto-maintained)
├── IMPROVEMENTS.md         # rule-gap log (self-improvement)
├── template.md             # session file template
├── README.md               # this file
└── YYYY-MM-DD/            # organized by date
    └── HH-MM-topic.md     # session records
```

## How It Works

This is an **automatic** system:

- **Auto-save**: The agent automatically saves when decision signals are detected - you don't need to ask
- **Auto-recall**: At the start of each session, the agent automatically reads recent decision records

## Self-Improvement Mechanism

If a decision should have been recorded but wasn't saved automatically, just say:

- "这条刚才没记下来" / "you didn't save that" / "漏了"

The agent will:
1. Immediately save the missed decision (recovery)
2. Record the gap in `IMPROVEMENTS.md` to improve trigger rules (prevention)

## Git Management

Recommended to commit this directory to the repository to support:
- Knowledge continuity across machines
- Decision sharing in team collaboration
- Complete project history

```bash
git add agent-memory/
git commit -m "docs: record AI session - [topic]"
```

## Notes

- The agent-memory system works across different AI tools (Claude, Cursor, Kiro, etc.)
- Session files are plain markdown - human-readable and tool-neutral
- Each session captures decisions, rationale, and lessons learned
