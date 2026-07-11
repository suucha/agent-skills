# Agent Memory

A skill that turns AI conversations into **reusable memory** — it auto-saves decisions when they happen and auto-recalls them next time, so no agent ever starts blind.

Unlike a passive archive, this system has two active mechanisms:

- **Auto-save** — detects decision signals and saves *without waiting for the user to ask*
- **Auto-recall** — reads prior decisions at the start of a session and surfaces them

The `agent-memory/` directory (plain markdown, committed to git) is the **provider-neutral memory layer**: each agent vendor locks memory inside its own format, this directory is the escape hatch that works across Claude Code, Cursor, opencode, Kiro, and any future agent.

## Features

- Injects auto-save + auto-recall rules into `AGENTS.md` and `CLAUDE.md`
- Scaffolds an `agent-memory/` directory and copies in the session template
- Supports a session format with multiple decision points
- Auto-saves on decision signals (no need to remember to ask)
- Auto-recalls prior decisions at session start
- Automatically sanitizes sensitive information
- Works for both coding and non-coding tasks
- Saves each session in the user's conversational language

## Install

```bash
npx skills add suucha/agent-skills --skill agent-memory -g -y
```

## Usage

### Step 1: Set up the memory system

Tell the AI:

```
Set up memory
```

Or:

```
设置记忆
```

The AI will automatically:
1. Create the `agent-memory/` directory
2. Copy `template.md` into `agent-memory/`
3. Add the tracking rules to `AGENTS.md`
4. Add the tracking rules to `CLAUDE.md`

### Step 2: Just work — the system runs automatically

Once set up, you don't need to do anything:

- **At the start of each session**, the agent reads prior decisions and carries them forward
- **When you make a decision**, the agent auto-saves it (you'll get a brief "saved X" confirmation)

You can still explicitly say "save this" or "记一下" if you want to force a save at a specific moment.

## Directory structure

```
agent-memory/
├── template.md        # session template
├── README.md          # usage instructions
└── YYYY-MM-DD/
    ├── HH-MM-topic.md
    └── ...
```

## Session template

The `template.md` file defines the standard session format. It captures:

- Background and goals
- Multiple decision points (problem, options, decision, rationale)
- Execution plan
- Related content (code changes, references)
- Lessons learned
- Follow-ups / TODOs

## Sensitive information handling

Before saving, the AI scans for and sanitizes:

| Type | Handling |
|------|----------|
| Passwords, keys, tokens | Removed entirely |
| Personal identifiable information | Masked |
| Internal addresses | Removed or replaced |
| Financial information | Removed entirely |

## Language

Each saved session is written in the **same language the user is conversing in** — Chinese conversation → Chinese session file, English → English, and so on — unless the user explicitly asks for a specific language, in which case their choice is honored. The record is never translated into another language on its own.

## Integrations

### Git

Session files should be committed to the repo to support team collaboration:

```bash
git add agent-memory/
git commit -m "docs: record AI session - [topic]"
```

### .gitignore

If sessions may contain sensitive information, exclude them:

```
agent-memory/
```

## License

MIT
