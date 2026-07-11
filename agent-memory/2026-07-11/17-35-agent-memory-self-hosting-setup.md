# agent-memory 自举验证环境搭建

- **日期**: 2026-07-11 17:35
- **参与者**: 用户 + Kiro
- **类型**: 设计 + 开发

## 背景

正在开发 agent-memory 技能，需要在开发过程中用这个技能本身来验证其功能（自举/dogfooding）。讨论如何设置自举验证环境，以及如何处理目录名、技能加载、文件同步等关键设计决策。

## 讨论要点和决策

### 1. 目录名从 ai-sessions 改为 agent-memory

**问题**: 原技能名为 ai-sessions-track，后改名为 agent-memory，存储目录是否也需要改名？

**选项**:
- 保持 ai-sessions：向后兼容，描述内容（会话记录）
- 改为 agent-memory：与技能名一致，强调"记忆"而非"会话"

**决策**: 改为 agent-memory

**理由**:
- 技能核心是"记忆"（auto-recall），不只是记录会话
- 目录名应体现目的：这是 agent 的记忆系统，不是会话日志
- 技能名和目录名一致，降低认知负担
- 支持未来扩展（如知识图谱、向量检索等）

---

### 2. 技能加载路径：.kiro/skills（非符号链接）

**问题**: 如何让 Kiro 加载本地正在开发的 agent-memory 技能？

**尝试的方案**:
- Junction/符号链接：`.kiro/skills/agent-memory` → `skills/agent-memory`
- 实际文件复制：复制整个目录到 `.kiro/skills/`

**决策**: 使用实际文件复制，不用符号链接

**理由**:
- 符号链接在 Kiro/Claude/opencode 中不工作（实测失败）
- 需要复制到三个位置：`.kiro/skills/`、`.agents/skills/`、`.claude/skills/`
- 虽然需要手动同步，但确保各工具都能正确加载

---

### 3. 技能文件同步规则写入 AGENTS.md

**问题**: 如何确保修改 skills/agent-memory/ 后同步到各加载目录？

**决策**: 在 AGENTS.md 中明确记录同步规则

**理由**:
- AGENTS.md 会自动加载到 agent context 中
- Agent 看到规则后，修改技能文件时会主动同步
- 提供单行命令方便手动同步
- 明确说明何时需要同步（编辑 SKILL.md、README.md 等）

---

### 4. IMPROVEMENTS.md 的双重角色澄清

**问题**: references/IMPROVEMENTS.md 包含历史 gap 记录，角色不清晰

**决策**: 
- `skills/agent-memory/references/IMPROVEMENTS.md` 是纯净模板（只有结构）
- `agent-memory/IMPROVEMENTS.md` 是运行时文件（实际记录 gap）

**理由**:
- 模板文件应该干净，只展示格式
- 历史 gap（GAP-001、GAP-002）已经修复并体现在 SKILL.md 中，无需保留在模板
- 运行时的 gap 记录在项目的 agent-memory/ 中，通过 git 跨机器共享

---

### 5. agent-memory/ 目录提交到 git（不放入 .gitignore）

**问题**: agent-memory/ 目录应该被 git 追踪吗？

**选项**:
- 排除：视为运行时数据，每个开发者独立
- 追踪：视为项目知识，跨机器共享

**决策**: 提交到 git，不放入 .gitignore

**理由**:
- 这是技能的核心价值：跨机器、跨 agent 的记忆延续
- Provider-neutral memory layer 需要通过 git 实现共享
- 团队协作时，所有人都能看到历史决策
- 这正是自举验证的意义：用技能记录技能自己的演进

---

### 6. 删除 dogfooding/ 目录，直接用技能自己的机制

**问题**: 是否需要单独的 dogfooding/ 目录记录验证日志？

**决策**: 删除 dogfooding/，完全依赖技能自己的 self-improvement loop

**理由**:
- 技能已设计了 self-improvement loop（IMPROVEMENTS.md 机制）
- 吃自己的狗粮：用技能的设计来验证技能本身
- 避免重复：不需要两套记录"漏记"的机制
- 真实场景：作为普通用户使用技能，发现的问题更真实

## 执行计划

1. ✅ 重命名目录：ai-sessions → agent-memory（全局替换）
2. ✅ 创建 .kiro/skills/agent-memory（实际复制，非符号链接）
3. ✅ 在 AGENTS.md 中添加同步规则
4. ✅ 清理 references/IMPROVEMENTS.md 为纯净模板
5. ✅ 初始化 agent-memory/ 系统（目录、模板、索引）
6. ✅ 更新 AGENTS.md 和 CLAUDE.md 加入 AI Session Tracking 规则
7. ✅ 从 .gitignore 移除 agent-memory/

## 相关内容

### 文件修改
- `skills/agent-memory/SKILL.md`: 添加 keywords 字段，全局替换目录名
- `skills/agent-memory/README.md`: 更新目录名和说明
- `skills/agent-memory/references/IMPROVEMENTS.md`: 清理为纯净模板
- `AGENTS.md`: 创建并添加同步规则 + AI Session Tracking 规则
- `CLAUDE.md`: 复制 AGENTS.md 内容
- `.gitignore`: 排除 .agents/、.kiro/、.claude/，但保留 agent-memory/

### Git 提交
- `refactor: rename ai-sessions to agent-memory for semantic clarity`
- `feat: add keywords to agent-memory skill frontmatter`
- `feat: add .kiro/skills junction for local skill loading`
- `feat: add AGENTS.md with skill synchronization workflow`
- `chore: remove dogfooding directory`
- `feat: initialize agent-memory system for self-hosting validation`
- `refactor: clean IMPROVEMENTS.md template to pure seed format`

## 经验总结

- **自举验证的正确姿势**: 不需要额外的"验证基础设施"，直接用技能自己的机制就是最好的验证
- **符号链接的坑**: Windows junction 在某些工具中不工作，实际文件复制更可靠
- **单一真实来源**: skills/agent-memory/ 是源码，其他位置是部署副本，通过明确的同步规则管理
- **模板要纯净**: references/ 中的模板文件应该只包含结构和说明，不包含实际数据
- **记忆需要共享**: agent-memory/ 必须提交到 git，才能实现跨机器的记忆延续

## 后续待办

- [ ] 重启 Kiro，验证技能是否正确加载
- [ ] 开始使用技能进行日常开发，观察 auto-save 和 auto-recall 是否正常工作
- [ ] 发现漏记时，记录到 agent-memory/IMPROVEMENTS.md
- [ ] 根据 IMPROVEMENTS.md 的 gap 记录改进 SKILL.md
- [ ] 验证跨机器的记忆延续是否工作
