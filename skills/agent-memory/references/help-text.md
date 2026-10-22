# On-demand Help Text

**What this skill does**: Captures decisions during our conversations and recalls them next session, so any agent picking up this project knows what was already decided and why — you don't re-litigate settled questions.

**You don't need to do anything special.** Auto-save and auto-recall run on their own. But one habit dramatically improves how well it works:

## Use affirmative language when you've decided ⚠️

This skill detects decisions from your language. **Ambiguous phrasing gets missed; strong signals get saved.**

✅ **Strong signals** (reliably saved):
- 中文：「就选 A」「用 A 方案」「确定了」「就这样定」「就这么干」
- English: "let's go with X", "that's the one", "finalize X", "decision made"

❌ **Weak signals** (often missed):
- 中文：「选 A 吧」「可能用 A」「先试试 A?」「感觉 A 好一点」「先用 A 看看」
- English: "maybe A", "I guess A", "let's try A?", "I think A"

**If you've made up your mind, say so plainly.** If you're still exploring, that's fine — just don't expect a save until you commit.

## Other things worth knowing

- **Decisions to NOT do something count too.** "不做 X，因为 Y" / "we're not doing X because Y" is a first-class decision — recording it prevents the same question coming back next session.
- **Manual triggers**:
  - "记一下" / "save this" / "record this" — force a save right now
  - "存个检查点" / "checkpoint this" / "记录讨论进度" — save an in-progress discussion snapshot (not yet a decision); recorded as `### checkpoint.` section, surfaced differently during recall
  - "漏了" / "这条刚才没记下来" / "you didn't save that" — flag a missed save (the system self-improves via `IMPROVEMENTS.md`)
  - "记忆帮助" / "memory help" — show this help again
- **Where things live**:
  - `.agent-memory/index.yaml` — session index (newest first)
  - `.agent-memory/YYYY-MM-DD/HH-MM-topic.md` — individual session files
  - `.agent-memory/IMPROVEMENTS.md` — backlog of missed saves (the system's own bug tracker)
