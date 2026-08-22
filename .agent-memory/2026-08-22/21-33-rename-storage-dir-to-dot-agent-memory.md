# 记忆存储目录改名 agent-memory → .agent-memory

- **Date**: 2026-08-22 21:33
- **Participants**: tubo + CodeBuddy
- **Type**: design

## Background

记忆系统的"存储记忆的文件夹"当前叫 `agent-memory/`(仓库根目录)。用户希望改用 dotfile 约定 `.agent-memory/`,把它从项目根目录默认视图里隐藏,明确这是 agent 管理的工具目录而非项目源码。同时需要明确:已安装用户当前是 `agent-memory/`,升级技能后旧目录如何处置。

## Discussion points and decisions

### 1. 存储目录改名 agent-memory → .agent-memory

**Problem**: 根目录下的记忆存储目录 `agent-memory/` 容易被误认为是项目源码/文档,且占用根目录默认视图。

**Decision**: 改名为 `.agent-memory/`(dotfile 约定),仅针对**数据存储目录**(根目录那个),不针对技能包目录。

**Rationale**:
- dotfile 约定把"工具/配置目录"与项目源码区分开,根目录更干净
- 目录内 `index.yaml` 的 `file` 字段是相对路径(`2026-07-12/...`),整体改名后无需改动
- 与已有 `skills/agent-memory/`(技能包)区分清晰,二者含义不同

---

### 2. 改名范围界定(明确不动什么)

**Problem**: 仓库里两处都叫 `agent-memory`,容易误改。

**Decision**: 仅改"数据存储目录 + 文档里的文本引用";技能包目录 `skills/agent-memory/` 及其三处同步副本(`.kiro/.agents/.claude/skills/agent-memory/`)**不动**;历史会话文件里的 `agent-memory` 字样**不回改**(属历史记录,git 已保留)。

**Rationale**:
- `skills/agent-memory/` 由 `SKILL.md` 的 `name: agent-memory` 标识,是技能安装位置,与"存储记忆的文件夹"无关
- 历史会话文件(如自举记录)描述的是过去的决策,回改会破坏历史准确性
- `.claude/settings.local.json` 里的 8 处引用全是技能包同步命令,亦不动

---

### 3. 升级迁移策略:确认后自动迁移(Option A)

**Problem**: 已安装用户当前是 `agent-memory/`,升级后旧目录怎么办——重命名、保留、还是双目录共存?

**Options**:
- A 确认后自动迁移:升级时若检测到 `agent-memory/` 且 `.agent-memory/` 不存在,先征询用户再 `git mv` + 改写 AGENTS.md/CLAUDE.md 引用
- B 双目录兼容:skill 自动读取"已存在的那个"目录并写回,两名字长期共存
- C 硬切换:只认 `.agent-memory/`,旧目录留给用户手动处理

**Decision**: 采用 **A 确认后自动迁移**。

**Rationale**:
- 全局单一规范名,避免生态里长期出现两个目录名
- 不静默搬用户数据——迁移前征询确认,降低风险
- 迁移在 init/升级时一次性完成,日常 save/recall 热路径永远只认 `.agent-memory/`,逻辑干净
- 双目录共存(B)会让热路径分支化、文档复杂;硬切换(C)会令未手动迁移的升级用户丢失旧记忆回想

---

### 4. 顺手修 evals.json 旧命名

**Problem**: `skills/agent-memory/evals/evals.json` 元数据仍是旧名(`skill_name: ai-session-tracker`、`ai-sessions` 目录),与实际技能名/目录不一致。

**Decision**: 借本次改名一并修正为 `agent-memory`。

**Rationale**: 避免评测与实际技能名脱节;顺手清理技术债。

## Execution plan

1. `git mv agent-memory .agent-memory`(数据目录整体改名,相对路径不受影响)
2. 在 SKILL.md 增加"升级迁移"逻辑:init 时检测旧 `agent-memory/` → 征询 → `git mv` + 改写 AGENTS.md/CLAUDE.md 块
3. 替换所有文档文本引用 `agent-memory/`(数据目录义)→ `.agent-memory/`(SKILL.md / README / references/README / AGENTS.md / CLAUDE.md / CONTRIBUTING.md / 根 README)
4. 重新同步三处副本(.kiro/.agents/.claude)
5. 修正 evals.json 的 `skill_name` 与 `ai-sessions` 引用
6. 历史会话文件不回改

## Related content

### Code / file changes
- `agent-memory/` → `.agent-memory/`:目录改名(git mv)
- `skills/agent-memory/SKILL.md`:新增升级迁移逻辑 + 目录名引用更新
- `AGENTS.md` / `CLAUDE.md`:canonical tracking block 中 `agent-memory/` → `.agent-memory/`
- `skills/agent-memory/evals/evals.json`:`ai-session-tracker`/`ai-sessions` → `agent-memory`

### References
- 历史决策:目录名从 ai-sessions 改为 agent-memory(2026-07-11 自举记录)—— 不回改

## Lessons learned

- 仓库内"数据目录"与"技能包目录"同名易混淆,改动前必须明确区分二者
- 相对路径的 index.yaml 让整体目录改名零成本
- 用户数据迁移必须"先确认再执行",不能静默搬

## Follow-ups / TODOs

- [ ] 在 SKILL.md 实现升级迁移的检测+征询+git mv 逻辑
- [ ] 重新同步 .kiro/.agents/.claude 三处副本
- [ ] 验证改名后自举(本仓库)记忆仍可正常 recall
