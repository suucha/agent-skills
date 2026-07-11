# Contributing to Suucha Agent Skills

Thanks for your interest in contributing! This document provides guidelines for contributing to the project.

## 🐛 Reporting Issues

If you find a bug or have a feature request:

1. **Check existing issues** first to avoid duplicates
2. **Open a new issue** with a clear title and description
3. For bugs, include:
   - Steps to reproduce
   - Expected vs actual behavior
   - Your environment (OS, AI agent, skill version)
4. For feature requests, explain:
   - The problem you're trying to solve
   - Why existing features don't address it
   - Your proposed solution (if any)

## 💡 Suggesting New Skills

If you have an idea for a new skill that fits the "agent memory/collaboration" theme:

1. Open an issue with the `skill-proposal` label
2. Describe:
   - What problem it solves
   - How it relates to existing skills
   - Why it belongs in this repo (vs. a separate repo)

## 🔧 Contributing Code

### Setup

1. Fork the repo
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/agent-skills.git
   cd agent-skills
   ```
3. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

### Making Changes

1. **For bug fixes:**
   - Add a test case that reproduces the bug (if applicable)
   - Fix the bug
   - Verify the test passes

2. **For new features:**
   - Discuss in an issue first (for larger changes)
   - Follow existing patterns (e.g., SKILL.md structure, README format)
   - Update documentation

3. **For documentation:**
   - Fix typos, improve clarity, add examples
   - Keep the tone consistent with existing docs

### Commit Guidelines

Use conventional commits format:

```
type(scope): description

[optional body]
[optional footer]
```

**Types:**
- `feat:` — new feature
- `fix:` — bug fix
- `docs:` — documentation changes
- `refactor:` — code refactoring (no behavior change)
- `test:` — adding or updating tests
- `chore:` — maintenance tasks

**Examples:**
```
feat(agent-memory): add support for decision tags
fix(agent-memory): handle edge case in auto-save checkpoint
docs(README): add installation troubleshooting section
refactor(agent-memory): simplify recall logic
```

### Pull Requests

1. Push your branch to your fork
2. Open a PR against `main`
3. Fill in the PR template:
   - What does this PR do?
   - Why is this change needed?
   - How was it tested?
   - Any breaking changes?
4. Link related issues (e.g., "Fixes #123")
5. Wait for review

**PR Guidelines:**
- Keep PRs focused (one feature/fix per PR)
- Update documentation if behavior changes
- Add tests for new features (if applicable)
- Ensure existing tests pass

## 📝 Skill Structure

If you're adding a new skill, follow this structure:

```
your-skill-name/
├── README.md                # User-facing docs
├── SKILL.md                 # Agent-facing instructions (with frontmatter)
├── evals/                   # Test cases (optional)
│   └── evals.json
└── references/              # Supporting files
    └── *.md
```

**SKILL.md frontmatter:**
```yaml
---
name: your-skill-name
description: Brief description (used in skill discovery)
---
```

## 🧪 Testing

For skills with `evals/`:
- Add test cases that verify key behaviors
- Test across different AI agents if possible
- Document any agent-specific quirks

## 📜 License

By contributing, you agree that your contributions will be licensed under the MIT License.

## ❓ Questions?

If you're unsure about anything:
- Open an issue with the `question` label
- Tag @suucha for clarification

---

**Thank you for contributing! 🎉**
