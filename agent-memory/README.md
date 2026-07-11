# Agent Memory Sessions

这个目录存储 AI agent 的记忆 - 开发 agent-memory 技能过程中的决策记录。

## 目录结构

```
agent-memory/
├── index.yaml              # 会话索引（自动维护）
├── IMPROVEMENTS.md         # 规则缺口日志（自我改进）
├── template.md             # 会话文件模板
├── README.md               # 本文件
└── YYYY-MM-DD/            # 按日期组织
    └── HH-MM-topic.md     # 会话记录
```

## 使用说明

这是一个**自动**系统：

- **Auto-save（自动保存）**: 当检测到决策信号时，agent 会自动保存，无需你主动要求
- **Auto-recall（自动回忆）**: 每次会话开始时，agent 会自动读取最近的决策记录

## 自我改进机制

如果某个决策应该被记录但没有被自动保存，只需说：

- "这条刚才没记下来"
- "you didn't save that"
- "漏了"

Agent 会：
1. 立即补记这个决策
2. 在 `IMPROVEMENTS.md` 中记录这个缺口，用于改进触发规则

## Git 管理

建议将这个目录提交到仓库，以支持：
- 跨机器的知识延续
- 团队协作时的决策共享
- 项目历史的完整记录

```bash
git add agent-memory/
git commit -m "docs: record AI session - [topic]"
```

## 关于这个项目

这个 `agent-memory/` 目录是 agent-memory 技能**自己**产生的 - 我们在开发这个技能的同时，用它来记录开发过程中的决策。这是最严格的测试：如果技能连自己的开发都支撑不了，怎么推荐给其他人？
