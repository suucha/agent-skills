# Dogfooding Log

这个目录记录 agent-memory 技能的自举验证过程。

## 验证方法

1. 通过 `.agents/skills/agent-memory` (junction) 让 Kiro 加载此技能
2. 在日常使用中依赖此技能自动记录决策到项目根目录的 `agent-memory/`
3. 发现问题时记录在此目录，然后改进 `SKILL.md`
4. 重启 Kiro 后改进立即生效

## 验证目标

验证成熟度标准：
- ✅ 连续 2 周无漏记案例
- ✅ `improvements.md` 清单全部实施
- ✅ 至少覆盖 20+ 个真实决策场景

## 记录类型

### missed-saves.md
记录"该记但没记"的案例。

**记录格式：**
```markdown
## YYYY-MM-DD HH:MM - 未记录"[决策类型]"

**场景：** [描述当时的讨论和决策]
**用户说法：** "[用户的原话]"
**为什么没记：** [分析触发信号或 self-check 的问题]
**改进方向：** [下一步要做的改进]
```

### false-positives.md
记录"不该记但记了"的案例。

**记录格式：**
```markdown
## YYYY-MM-DD HH:MM - 误记录"[内容]"

**场景：** [描述什么被误记录了]
**触发原因：** [哪个信号被误判]
**改进方向：** [如何收紧触发条件]
```

### improvements.md
从案例中提炼的改进清单（待办）。

**记录格式：**
```markdown
- [ ] [改进项描述] (来源: missed-saves.md #链接 或 false-positives.md #链接)
- [x] [已完成的改进] (YYYY-MM-DD 实施)
```

## 工作流程

### 发现漏记时

1. 你说："这条刚才没记下来"
2. Kiro 手动补记到 `agent-memory/`
3. **同时** Kiro 在 `dogfooding/missed-saves.md` 追加案例
4. 定期回顾 → 分析模式 → 更新 SKILL.md

### 发现误记时

1. 你说："这个不需要记"
2. Kiro 在 `dogfooding/false-positives.md` 追加案例
3. 定期回顾 → 收紧触发条件 → 更新 SKILL.md

### 改进循环

```
使用 → 发现问题 → 记录到 dogfooding/ 
     → 更新 SKILL.md → 重启 Kiro → 改进生效
     → 继续使用 → ...
```

## 元价值

这个自举过程本身展示了：
- ✅ AI 技能的正确开发方式：边用边改
- ✅ 自举是最严格的测试：连创建者都依赖得上
- ✅ 知识的复利：每个改进立即体现在工具里

这个目录将成为技能可信度的最佳证明。
