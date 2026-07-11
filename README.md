# Suucha Agent Skills

A collection of AI agent skills focused on memory, collaboration, and cross-agent workflows.

## 🎯 Vision

Modern development involves multiple AI agents (Claude Code, Cursor, Kiro, OpenCode, etc.), but each agent starts every session blind — no memory of prior decisions, no context from previous conversations. This toolkit fixes that by building a **provider-neutral memory layer** that works across all agents.

## 📦 Available Skills

### 🧠 [agent-memory](./agent-memory/)

Turn AI conversations into **reusable memory** — auto-saves decisions when they happen and auto-recalls them next time, so no agent ever starts blind.

**Key features:**
- Auto-save decisions without waiting for you to ask
- Auto-recall prior decisions at session start
- Provider-neutral (works across Claude Code, Cursor, Kiro, OpenCode, etc.)
- Sanitizes sensitive information automatically
- Multi-language support

**Install:**
```bash
npx skills add suucha/agent-skills --skill agent-memory -g -y
```

**Use case:** Project memory, cross-agent knowledge retention, decision tracking

---

### 🚧 Coming Soon

- **memory-sync** — Real-time memory sync across different agents and machines
- **session-replay** — Replay and debug agent conversations
- **decision-viz** — Visualize decision history and dependencies

---

## 🚀 Quick Start

### Installation

All skills can be installed via the `skills` CLI:

```bash
# Install a specific skill globally
npx skills add suucha/agent-skills --skill <skill-name> -g -y

# Example: install agent-memory
npx skills add suucha/agent-skills --skill agent-memory -g -y
```

### Usage

Once installed, activate the skill in your AI chat:

```
Set up session tracking
```

The agent will automatically configure the memory system for your project.

---

## 🏗️ Repository Structure

```
suucha/agent-skills/
├── README.md                    # This file
├── LICENSE                      # MIT
├── agent-memory/               # Memory system skill
│   ├── README.md
│   ├── SKILL.md
│   ├── evals/
│   └── references/
└── [future-skills]/            # More skills coming soon
```

---

## 🤝 Contributing

Contributions are welcome! Whether it's:

- 🐛 Bug reports
- 💡 Feature requests
- 📝 Documentation improvements
- 🔧 Code contributions

Please open an issue or PR. See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

---

## 📄 License

MIT © [Suucha](https://github.com/suucha)

---

## 🌟 Why This Exists

**The problem:** You have a great conversation with Claude Code on Monday, make key decisions. On Tuesday, you open Cursor — it knows nothing. You switch machines — the new agent starts from zero. Every session is a blank slate.

**The solution:** A provider-neutral memory layer that sits in your repo (plain markdown + yaml), committed to git, that every agent can read and write to. Decisions made in one agent are automatically available to the next.

This is the memory system I built for my own projects after getting tired of re-explaining the same decisions to different agents. Now I'm sharing it.

---

## 📚 Learn More

- [agent-memory documentation](./agent-memory/README.md)
- [Session file format](./agent-memory/references/template.md)
- [Design philosophy](./agent-memory/references/IMPROVEMENTS.md)

---

**Made with ❤️ for developers who work with multiple AI agents**
