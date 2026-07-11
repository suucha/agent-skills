# Agent Instructions

This project develops AI agent skills, specifically the `agent-memory` skill.

## Development Workflow — Skill File Synchronization

When developing the `agent-memory` skill, files in `skills/agent-memory/` serve as the **single source of truth**.

### After modifying any file in `skills/agent-memory/`:

**ALWAYS synchronize changes to all skill directories:**

```bash
# Copy to Kiro
Copy-Item -Recurse -Force "skills\agent-memory" ".kiro\skills\agent-memory"

# Copy to opencode (Agents)
Copy-Item -Recurse -Force "skills\agent-memory" ".agents\skills\agent-memory"

# Copy to Claude Code
Copy-Item -Recurse -Force "skills\agent-memory" ".claude\skills\agent-memory"
```

**Or use a single command to sync all:**

```bash
Copy-Item -Recurse -Force "skills\agent-memory" ".kiro\skills\agent-memory"; Copy-Item -Recurse -Force "skills\agent-memory" ".agents\skills\agent-memory"; Copy-Item -Recurse -Force "skills\agent-memory" ".claude\skills\agent-memory"
```

### Why this is necessary:

- **Junction/symlinks don't work** with Kiro/Claude/opencode skill loading
- **Real file copies are required** for each tool to recognize the skill
- **Three copies** ensure the skill works across all agent platforms
- **Manual sync** is needed because file watchers don't trigger on source changes

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
