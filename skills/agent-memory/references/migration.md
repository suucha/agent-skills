# Upgrade Migration — from old `agent-memory/` directory

The storage folder was renamed from `agent-memory/` to `.agent-memory/` (dotfile convention). Existing installs still have `agent-memory/`. When this skill runs init / recall and finds `agent-memory/` present but `.agent-memory/` absent, migrate it so the project converges on the single canonical name.

## Flow (confirm before moving user data — never silent)

1. **Detect**: `agent-memory/` exists AND `.agent-memory/` does NOT exist.

2. **Tell the user**: "检测到旧记忆目录 `agent-memory/`,是否迁移到 `.agent-memory/`?(旧目录会被重命名,历史由 git 保留)"

3. **On confirmation, migrate**:
   - In a git repo: `git mv agent-memory .agent-memory` (preserves history)
   - Not a git repo: `mv agent-memory .agent-memory`
   - Rewrite references in `AGENTS.md` and `CLAUDE.md`: replace `agent-memory/` → `.agent-memory/` (data-dir references only — do NOT touch `skills/agent-memory`, the skill package path)

4. **If BOTH `agent-memory/` and `.agent-memory/` exist**: prefer `.agent-memory/`; tell the user the old `agent-memory/` is stale and can be deleted after they confirm nothing is missing.

5. **After migration**, proceed with `.agent-memory/` for all subsequent save/recall. The recall/save hot path always uses `.agent-memory/` — there is no dual-directory fallback.

## Why confirm first

Moving a user's memory is a data operation. The skill must not silently relocate files; the user confirms, then the one-time move happens.
