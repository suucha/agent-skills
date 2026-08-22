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

---

### 7. 初始化触发词改为记忆相关术语

**问题**: 原触发词 "set up session tracking" 偏向会话跟踪/日志记录，不够体现"记忆"概念

**选项**:
- 保持 "session tracking": 向后兼容，但概念不准确
- 改为 "memory" 相关: "set up memory"、"initialize memory"、"enable memory"

**决策**: 使用 "set up memory" / "initialize memory" / "enable memory" 以及对应的中文版本

**理由**:
- 技能核心是"记忆"（auto-recall），不只是记录会话
- 更直观、更自然的用户表达方式
- 降低认知负担（技能名 = 触发词概念）
- 支持中英文："设置记忆"、"初始化记忆"、"启用记忆"

---

### 8. 语言规则明确跨国团队场景

**问题**: 原语言规则已支持用户指定语言，但没有明确说明典型场景（跨国团队）

**决策**: 重构语言规则章节，分为"默认行为"和"用户指定"两部分，明确举例跨国团队用例

**理由**:
- 技能本身保持英文通用性
- 会话记录根据用户语言或明确指定
- 常见场景：中文对话但记录用英文（便于国际团队协作）
- 结构化展示更清晰

---

### 9. README.md 模板化（修正设计缺陷）

**问题**: 
- `references/` 目录有 `template.md` 和 `IMPROVEMENTS.md`，但缺少 `README.md`
- 初始化时 `agent-memory/README.md` 是即兴生成的中文内容，不是从模板复制
- 三个文件来源不一致

**决策**: 创建 `references/README.md` 英文模板，初始化时统一从 `references/` 复制三个文件

**理由**:
- 保持一致性：三个初始化文件都应该来自模板目录
- 确保语言：默认英文（通用性），不依赖即兴生成
- 便于维护：修改模板一处，所有新初始化的项目都受益
- 文档质量：模板经过打磨，优于即兴内容

## 执行计划（续）

8. ✅ 更新触发词：SKILL.md、README.md（从 "session tracking" 改为 "memory" 系列）
9. ✅ 优化语言规则：SKILL.md、AGENTS.md、CLAUDE.md、README.md（明确跨国团队场景）
10. ✅ 创建 references/README.md 模板
11. ✅ 更新 SKILL.md 初始化步骤（明确三个文件都从 references 复制）

## 相关内容（续）

### 新增 Git 提交
- `refactor: update initialization trigger words to memory-focused terms`
- `docs: clarify language handling with cross-border team examples`
- `docs: translate agent-memory/README.md to English for universality`
- `feat: add README.md template for agent-memory initialization`

## 经验总结（续）

- **触发词的语义匹配很重要**: 技能名和触发词应该在概念上一致，降低用户的认知负担
- **文档结构化展示**: 将复杂规则分为"默认行为"和"特殊情况"，更易理解
- **模板文件要完整**: 所有初始化时需要的文件都应该在 references/ 中准备好，避免即兴生成导致的不一致

- **决策记录的粒度权衡**: 
  - 同一主题的多个小决策可以追加到同一会话文件
  - "讨论式"决策比"指令式"决策更难被自动触发
  - 自举验证时应该更积极地记录，以测试系统边界

## 后续待办（更新）

- [ ] 重启 Kiro，验证技能是否正确加载
- [ ] 测试新的触发词是否工作（"set up memory" / "设置记忆"）
- [ ] 开始使用技能进行日常开发，观察 auto-save 和 auto-recall 是否正常工作
- [ ] 发现漏记时，记录到 agent-memory/IMPROVEMENTS.md（特别关注"讨论式"决策的触发问题）
- [ ] 根据 IMPROVEMENTS.md 的 gap 记录改进 SKILL.md
- [ ] 验证跨机器的记忆延续是否工作


---

### 10. 决策记录粒度策略：追加模式

**问题**: 后续讨论了 3 个小决策（触发词、语言规则、README 模板），是否应该记录？粒度太细会不会有问题？

**选项**:
- 方案 A：追加到当前会话 - 同一主题的延续，保持上下文完整
- 方案 B：不记录 - 认为太细，git commit 够了
- 方案 C：独立新会话 - 单独文件记录

**决策**: 选择方案 A，追加到当前会话

**理由**:
- 这些是同一主题（完善技能设计）的延续
- 上下文完整：和前面的决策在同一个"自举搭建"会话中
- 便于回顾：一个文件看完整个搭建过程
- 测试追加模式：验证同一会话多次追加的可行性
- 自举验证关键时刻：如果连明确讨论的改动都不记，未来怎么说服别人用这个技能？

**元记录 - 这个决策本身的漏记**:
- 用户说"那就选1"时，agent 应该**先记录这个选择的决策**，再执行追加任务
- 但 agent 直接执行了任务，漏记了"选择方案1"这个决策本身
- 这是 SKILL.md 中明确规定的：decision + task → save FIRST, then execute
- 这个漏记现在正在补救中（本条记录）
- Gap 已记录到 IMPROVEMENTS.md

---

### 11. 新增"记忆帮助"触发词 + 帮助文本（强调肯定语气）

**问题**:
- 用户如何知道技能做什么、怎么触发？
- 更重要的：模糊语气（"选A吧"/"可能用A"）是漏存的主因之一，如何引导用户用肯定语气？

**选项**:
- 方案 A：init 完成时自动弹帮助 — 每次开 session 都重复，容易惹烦，与"无需用户主动"理念冲突
- 方案 B：被动触发（用户说"记忆帮助"时显示）+ init 不弹 — 安静，需要时才有
- 方案 C：不做 — 用户既然主动初始化，基本知道

**决策**: 选择方案 B — 被动触发，init 不弹；帮助文本里重点强调"用肯定语气"这条 tip

**理由**:
- "肯定语气" tip 是最有价值的部分——直接降低 IMPROVEMENTS.md gap 数量（模糊语气是漏存主因之一）
- "做什么/怎么触发"价值次要，用户主动初始化基本知道
- init 弹窗与技能"无需用户主动触发"的设计理念冲突，每次开 session 重复会惹烦
- 被动触发 = 安静 + 需要时就有，符合技能风格
- 不修改 trigger signals 表（保持严格，避免对探索性讨论过度触发）；改为通过帮助文本教育用户

**执行计划**:
1. SKILL.md 的 description 加 help 触发词（"memory help"/"记忆帮助"/"how does memory work" 等）
2. SKILL.md 新增 "## On-demand help" 章节，含帮助文本
3. 帮助文本重点突出"肯定语气"对比（强信号 vs 弱信号示例）
4. 同步到 .kiro/、.agents/、.claude/ 三个目录

---

### 12. 修复 CLAUDE.md / AGENTS.md 同步命令的嵌套目录 bug

**问题**: 文档化的同步命令 `Copy-Item -Recurse -Force "skills\agent-memory" ".kiro\skills\agent-memory"`(以及 bash 下对应的 `cp -r`)在目标目录已存在时，会把源目录**复制进**目标目录（形成 `.kiro/skills/agent-memory/agent-memory/` 嵌套），而不是替换目标。本次同步已经踩过这个坑——三个目标目录都被嵌套了，靠 `rm -rf` 再 `cp` 才修复。

**根因**: `Copy-Item -Recurse` 和 `cp -r` 都遵循 POSIX 语义：dest 存在时，把 source 作为 source-name 子目录放进 dest。这不是 bug 是设计，但对文档化"同步"工作流来说是隐式陷阱——首次执行后 dest 永远存在，所有后续同步都中招。

**选项**:
- 方案 A：保持现状（`Copy-Item -Recurse -Force`），加文档警告 — 警告容易被忽略，bug 还在
- 方案 B：改为 delete-then-copy 模式（`Remove-Item -Recurse -Force` + `Copy-Item -Recurse`）— 显式、可靠、跨平台
- 方案 C：用 `robocopy /MIR` — Windows 原生、快，但跨平台不友好
- 方案 D：写同步脚本（PowerShell 或 git hook）— 治本但增加维护负担

**决策**: 方案 B — 改为 delete-then-copy；同时修正代码块的语言标签（原写 `bash` 但内容是 PowerShell，误导）；提供 PowerShell 和 bash 两版命令（agent 运行环境可能不同）。

**理由**:
- 显式清理是最可靠的跨平台方式，无需依赖工具特定语义
- 比 robocopy 更可移植（robocopy Windows 限定）
- 比脚本化更简单（修改门槛低，agent 一次就能执行对）
- 双语版（PowerShell + bash）覆盖两种 agent shell 场景

**执行**:
1. CLAUDE.md 第 5-48 行的同步章节整体替换为 delete-then-copy 版本
2. AGENTS.md 第 5-48 行同样替换
3. 代码块语言标签从 `bash` 改为 `powershell`（PowerShell 命令）和 `bash`（bash 命令）
4. 解释为什么需要 delete-then-copy（一句话，避免后人误改）


---

### 13. 会话文件结构优化方案确定

**问题**: 当前"一天一个文件"模式边界不清晰；如果每会话一个文件，长会话后面能否记住保存到哪个文件；如果 agent 自动检测主题切换新建文件，可能文件数过多。

**讨论的选项**:
- 方案 A：严格"每会话一个文件" — 需要 agent 记住上下文，长会话可能遗忘
- 方案 B：agent 自动检测主题切换 — 可能过度分割，读最新 5 个文件不够
- 方案 C：混合策略（主会话 + 子话题章节）— 用 `###` 分隔小主题，一个文件内多个决策点

**用户进一步建议**: 为了 agent 读取方便，是否应该改用 JSON 格式 + 提供读取/操作脚本？

**决策**: 拒绝 JSON 方案，保持 Markdown 格式，选择方案 C（混合策略），通过标准化结构和明确规则解决 agent 解析问题

**理由**:
- **保留核心价值**: Provider-neutral（纯文本）、人类可读（Markdown 比 JSON 友好）、Git 友好（易于 diff）
- **JSON + 脚本会破坏这些价值**: 需要依赖脚本（不再零依赖），人类阅读困难，增加维护负担
- **Markdown 的解析问题可以通过规则解决**: 标准化结构 + index.yaml 增强 + 明确提取规则
- **方案 C 平衡了所有需求**: 
  - 章节分隔对人类友好（易读）
  - 章节标题（`###`）对 agent 友好（精确定位）
  - 一个文件内多个决策避免文件碎片化

**具体改进方案**:

1. **标准化 Markdown 结构（严格格式）**:
   - 在 SKILL.md 中新增 "Session File Structure (strict format)" 章节
   - 每个 `###` 标题 = 一个小主题/决策点
   - 每个小主题必须包含 `**Problem:**`、`**Decision:**`、`**Rationale:**` 三个字段
   - 章节顺序固定，agent 可以精确定位

2. **index.yaml 增强**:
   - 新增 `subTopics` 字段（从文件的 `###` 标题自动提取）
   - 新增 `lastUpdated` 字段（判断是否追加到同一文件）
   - Auto-recall 时可以快速浏览所有小主题，无需通读全文

3. **Auto-recall 明确提取规则**:
   - 更新 SKILL.md 的 Step 2，增加精确的解析指令
   - 读取 index.yaml → 查看 subTopics → 打开文件提取 `###` 章节
   - 每个章节提取 Problem/Decision/Rationale
   - 忽略低优先级章节（Execution plan、Related content、Follow-ups）

4. **追加逻辑明确化**:
   - 更新 SKILL.md 的 Step 3.3，明确何时追加、何时新建
   - 基于 `lastUpdated` 判断：最近 3 小时内 = 追加，否则新建
   - 追加时更新 index.yaml 的 `decisions` 计数、`lastUpdated`、`subTopics`

5. **Template 同步修改**:
   - 在 `skills/agent-memory/references/template.md` 顶部增加结构说明
   - 同步修改 `agent-memory/template.md`

**不包含在方案中的**:
- ❌ 不需要 `status` 字段（用 `lastUpdated` 即可）
- ❌ 不需要读取/操作脚本（保持零依赖）
- ❌ 不改为 JSON 格式（保留 Markdown 核心优势）

**执行计划**:
1. 修改 `skills/agent-memory/SKILL.md`（新增结构章节、更新 Auto-recall/Save flow、更新 Canonical tracking block）
2. 修改 `skills/agent-memory/references/template.md`（顶部增加格式说明）
3. 修改 `agent-memory/template.md`（同步 references 版本）
4. 修改 `AGENTS.md`（同步 Canonical tracking block）
5. 修改 `CLAUDE.md`（同步 AGENTS.md）
6. 同步到 .kiro/skills、.agents/skills、.claude/skills 三个目录
7. 提交 git

---

### 14. 1/2/3 点根因澄清：自举时序问题

**问题**: 审查决策 #13 落地情况时发现三个表面 gap：(1) 字段名语言矛盾（SKILL.md 要求 `**Problem**` 但实际文件用 `**问题**`），(2) index.yaml 缺少 `subTopics` 字段，(3) 缺少一致性验证。这三个是独立的设计缺陷吗？

**决策**: 三个点是同一个根因——自举时序问题。"先记录决策，再执行"规则导致上一个 agent 记录决策 #13 时技能还没修改，所以按旧规则保存了。不是设计缺陷，而是一次性的自举产物。

**理由**:
- "save first, then execute" 规则在修改技能本身时产生悖论：保存决策 #13 用的还是旧技能
- 旧格式记录只存在于已保存的历史会话中，新会话会用新格式
- 真正待决定的是：是否回填旧记录到新格式（open question）

---

### 15. 3 小时追加窗口降级为 hint，主信号用主题连续性

**问题**: 决策 #13 把"3 小时内追加"当成硬规则。但时间启发式既会误合并（2 小时间隔但不同主题），也会误拆分（4 小时同主题长会话）。

**决策**: 把"3 小时"降级为辅助 hint，主信号改用主题连续性判断。

**理由**:
- 时间是 fuzzy 代理指标，主题连续性才是本质信号
- 显式"新话题"触发词（如"换个话题"/"new topic"）作为补充信号——是否加入待确认

---

### 16. Auto-recall 分两层：轻量 index 扫描 + 深度文件读取

**问题**: 当前 auto-recall 无脑读最近 5 个 session 文件、逐个提取 Problem/Decision/Rationale。规模增长（100+ session）后会慢且漏。

**决策**: auto-recall 分两层——
- 轻量层：只读 index.yaml，扫所有 session 的 subTopics（判断哪些和当前问题相关）
- 深度层：只打开 subTopics 匹配当前问题的那几个文件

**理由**:
- 避免无脑读 5 个文件，按相关性按需深入
- index.yaml 已有 subTopics 字段（决策 #13），充分利用
- 轻量层成本极低，可以每次都跑

---

### 17. 关闭 GAP-003

**问题**: GAP-003（decision+task 顺序规则未执行）在 IMPROVEMENTS.md 中仍为 open 状态。

**决策**: 关闭 GAP-003——用户实测确认规则已执行。

**理由**:
- 规则已写入 SKILL.md（Step 3.1b）
- 用户测试验证通过
- 本次会话本身也在执行该规则（先保存决策 14-17，再执行后续任务）

---

### 18. 字段名强制英文

**问题**: 决策字段名语言矛盾——SKILL.md 严格要求 `**Problem**`/`**Decision**`/`**Rationale**`，但 template.md 的 Language note 说可以翻译成会话语言。#16 深度层解析需要明确规则。

**决策**: 字段名强制英文（`**Problem**` / `**Options**` / `**Decision**` / `**Rationale**`），内容可用中文。template.md 删除"可翻译字段名"的 Language note。

**理由**:
- 解析最简单可靠——agent 只需找固定的英文字段名
- 字段名是结构标记，不是内容，英文不碍阅读
- 跨语言团队一致——无论会话用什么语言，字段名统一
- 章节标题（`### N. [title]`）和正文内容仍可用会话语言

---

### 19. 旧记录不回填

**问题**: 现有会话文件 13 个旧决策用 `**问题**` 格式保存（自举时序产物）。是否迁移到新格式？

**决策**: 不回填。历史记录保持原样，新格式只对新会话生效。

**理由**:
- 接受自举产物——这是"先保存再执行"规则的自然结果
- 历史记录改动成本高、收益低
- 新会话自然用新格式，旧记录逐渐被新记录覆盖
- index.yaml 的 subTopics 已回填（只是标题，不是内容），足够支持 #16 轻量层

---

### 20. #15 不加显式"新话题"触发词

**问题**: 实现 #15（3 小时降级）时，是否加"换个话题"/"new topic"作为显式新会话触发词？

**决策**: 不加。只靠主题连续性判断是否新建会话文件。

**理由**:
- 规则简单——少一条规则少一份维护成本
- 主题连续性是本质信号，够用
- 如果未来发现误判多，再加显式触发词
