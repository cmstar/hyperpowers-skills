---
name: writing-skills
description: 使用 fresh-context RED–GREEN–REFACTOR 方法创建、更新和验证平台无关的 Agent Skill。仅当用户明确要求使用 writing-skills，或明确选择写有该名称的选项时使用；不要把否定、询问、文档提及或任务匹配视为调用授权。
---

# 编写 Skills

## 概述

把 RED–GREEN–REFACTOR 应用于 Agent 行为文档：

1. 先观察没有新 guidance 时的 baseline。
2. 编写解决真实失败的最小 Skill。
3. 用隔离上下文样本验证行为改变。
4. 根据新暴露的漏洞继续收紧。

核心原则：

> 如果没有观察到 baseline failure，就无法证明新 Skill 解决了正确的问题。

本技能独立运行，不要求调用任何其他技能。它与 TDD 共享方法论，但不会自动调用 `test-driven-development`。

## 激活边界

只有用户肯定地要求使用 `writing-skills`，或明确选择写有该技能名称的选项时才能调用。否定使用、询问功能，或任务涉及 `SKILL.md`、Agent instructions、技能验证，都不构成调用授权。

## 找到目标 Skill 目录

不要假定单一 runtime 路径：

1. 优先使用用户明确给出的目录。
2. 在仓库中工作时，检查项目已有的 Skill root。
3. 使用当前 Agent 暴露的 skill roots 或插件目录。
4. `~/.agents/skills/`、`~/.claude/skills/`、`~/.codex/skills/` 等只能作为候选，必须先验证。

使用 forward slashes 编写 Skill 内的相对路径。新 Skill 应自包含，不能依赖机器上的偶然绝对路径。

## 什么值得成为 Skill

适合：

- 会跨项目重复使用的 technique、pattern 或 reference；
- Agent 容易遗漏、误用或在压力下绕过的方法；
- 需要特定 workflow、工具或领域知识的任务。

不适合：

- 一次性解决方案；
- 项目专属 conventions（更适合项目 instructions）；
- 已经由标准工具充分解决的机械约束；
- 只记录某次会话经过的叙事文档。

## Skill 类型

| 类型 | 目的 | 主要验证方式 |
|---|---|---|
| Discipline | 在压力下强制关键规则 | pressure scenarios |
| Technique | 教会可复用的方法 | application 与 edge cases |
| Pattern | 建立判断模型 | recognition 与 counter-examples |
| Reference | 提供可检索信息 | retrieval 与正确应用 |

## Activation Policy

### 默认：Explicit activation

除非用户明确要求自动发现，新 Skill 默认只允许显式触发。

description 应表达：

```yaml
description: <一句话说明 Skill 的功能>。仅当用户明确要求使用 <skill-name>，或明确选择写有该名称的选项时使用；不要把否定、询问、文档提及或任务匹配视为调用授权。
```

显式触发包括：

- 用户直接说出 Skill 名称并要求使用；
- 用户明确选择一个写有 Skill 名称的编号或模式。

不包括：

- 任务内容高度匹配；
- 用户只说“继续”“开始吧”；
- plan、spec 或其他文档提到某 Skill；
- Agent 自己认为该 Skill 可能有帮助。

必须为显式激活 Skill 编写正向和负向行为测试。见 [examples/explicit-activation-testing.md](examples/explicit-activation-testing.md)。

### 可选：Automatic discovery

只有用户明确要求新 Skill 支持自动发现时，才使用 trigger-rich description：

- 描述具体 symptoms、situations 和 contexts；
- 包含 Agent 会搜索的术语；
- 描述何时加载，不要把完整 workflow 压缩进 description；
- 使用负向场景验证不会过度触发。

不得为了“更容易发现”而静默把显式激活 Skill 改成自动发现型。

## 目录结构

最小结构：

```text
skill-name/
└── SKILL.md
```

需要 progressive disclosure 时：

```text
skill-name/
├── SKILL.md
├── reference.md
├── examples/
│   └── focused-example.md
└── scripts/
    └── deterministic-tool
```

规则：

- SKILL.md 放 overview、决策边界和主 workflow。
- 100+ 行的 heavy reference 放单独文件。
- 可复用且确定性的工具才放 scripts。
- 所有 supporting files 都从 SKILL.md 直接链接，避免多层引用。
- 不为“看起来完整”添加无人使用的文件。

平台无关的结构与可移植性规则见 [agent-skills-best-practices.md](agent-skills-best-practices.md)。

## Frontmatter

最低要求：

```markdown
---
name: skill-name
description: [activation policy]
---
```

- `name` 使用小写字母、数字和连字符。
- `description` 保持单行、具体、可测试。
- 总长度遵循目标 runtime 和 Agent Skills specification 的限制。
- 不添加目标平台不支持且没有降级方案的字段。

## 编写 workflow

### RED：先建立 baseline

编辑 Skill 前：

1. 明确希望改变的 Agent 行为。
2. 创建能诱发失败的真实场景。
3. 在没有新 guidance 的隔离上下文中运行。
4. 逐字记录选择、遗漏和 rationalizations。

对于 discipline Skill，组合 time、authority、sunk cost、exhaustion 等压力。对于输出形状、technique 或 reference，使用与失败类型匹配的测试。

完整方法见 [testing-skills.md](testing-skills.md)。

### GREEN：编写最小 Skill

- 只解决已经观察到的失败。
- 选择适合失败类型的 guidance：
  - 明知规则却绕过 → 明确 prohibition、red flags、rationalization counters；
  - 输出形状错误 → positive recipe 或固定 contract；
  - 遗漏字段 → structural template；
  - 条件行为 → observable predicate。
- 给出一个高质量、可运行或可直接应用的 example。
- 不把 Agent 已经知道的常识写成长篇背景。

只有在 discipline Skill 的 baseline 显示 Agent 明知规则却仍在压力下绕过时，才参考 [persuasion-principles.md](persuasion-principles.md) 选择最小、合乎伦理的行为强化方式。

### VERIFY GREEN

使用相同场景和隔离方式重新运行：

- Agent 是否改变了选择？
- 是否正确应用 Skill，而不只是复述？
- 是否出现新的 rationalization？
- 输出是否稳定收敛？

失败时修改 Skill，而不是修改场景以制造通过。

### REFACTOR

- 删除没有行为价值的内容。
- 把重型细节移到直接链接的 reference。
- 为真实出现的新漏洞添加最小 counter。
- 重新运行已通过场景，防止回归。

## 隔离上下文行为测试

测试样本必须尽量不知道作者意图和先前结果。可使用：

- 当前平台原生的 fresh sub-agent；
- 新会话；
- 独立 API/eval harness；
- 其他能提供全新上下文的运行方式。

sub-agent 只是实现手段，不是必需的开发模式，也不构成对任何调度 Skill 的调用。

如果没有任何隔离运行能力：

- 可以按用户要求保留草稿；
- 必须明确标记“尚未完成行为验证”；
- 不得声称 Skill 已验证或可部署。

对 behavior-shaping wording：

- 保留 no-guidance control；
- 每个关键 variant 使用 5+ 个独立样本；
- 手工检查被自动规则标记的结果；
- 把 variance 视为质量信号。

## Cross-references

Skill 之间默认解耦：

- 不把其他 Skill 标记为必需依赖。
- 不要求打开另一个 Skill 才能理解当前 Skill。
- 可说明概念关系，但不能自动调用。
- 可向用户建议另一个 Skill；只有用户明确选择后才调用。
- 引用 supporting file 时使用普通相对链接，不使用会强制预加载的语法。

## 可视化

只有决策关系确实难以用短文本表达时才使用图形。优先使用当前 Agent 的原生 Mermaid、HTML、SVG 或其他可视化能力；无法可视化时使用 ASCII。不要强制特定 runtime、图形工具或渲染脚本。

## 常见错误

- 先写 Skill，再想办法证明它有效；
- 用学术问答代替真实行为场景；
- description 允许自动触发，却没有过度触发测试；
- 把 workflow 写进 description，导致 Agent 不读正文；
- 通过大量 prohibition 修复本应使用 positive recipe 的输出形状；
- supporting files 多层嵌套；
- 用多个平庸 examples 代替一个好 example；
- 没有验证当前 runtime 能否访问所引用文件或工具；
- 为完成 checklist 而生成无价值脚本。

## 完成检查

### RED

- [ ] 已定义要改变的行为。
- [ ] 已运行没有新 guidance 的 baseline。
- [ ] 已记录真实失败或确认没有问题可修。

### GREEN

- [ ] `name`、`description` 和目录结构合法。
- [ ] activation policy 与用户要求一致。
- [ ] 内容只处理实际失败。
- [ ] supporting files 均直接链接且存在。
- [ ] 已在相同场景中验证新 Skill。

### REFACTOR

- [ ] 已处理新出现的 rationalizations。
- [ ] 已重跑关键场景。
- [ ] 已检查过度触发、漏触发和路径可移植性。

### 完成状态

写完并验证后：

- 报告创建或修改的文件；
- 报告 baseline 与验证结果；
- 明确说明当前内容未提交；
- 不执行 `git add`、`git commit`、`git push` 或创建 PR；
- 不显示提交选项。

版本控制必须由用户在本技能结束后自行处理。
