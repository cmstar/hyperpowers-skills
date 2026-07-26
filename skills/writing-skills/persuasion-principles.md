# Skill 设计中的 Persuasion Principles

## 目的

纪律型 Skill 有时需要抵抗 Agent 在压力下的 rationalization。Persuasion principles 只应用于帮助 Agent 遵守用户认可的质量或安全规则，不能用于制造虚假紧迫感或扩大授权。

## 常用原则

### Authority

用明确、不可误解的语言表达真正不可协商的规则：

```text
Write code before the failing test? Remove only the current task's premature
implementation and restart from RED.
```

适用于 safety 或 discipline。普通 guidance 不要滥用绝对语气。

### Commitment

让已经显式激活的 workflow 保持一致：

- 开始时说明当前采用的模式；
- 对多步骤流程使用 checklist；
- 让用户明确选择互斥选项。

Commitment 不能用于自动激活 Skill。用户没有显式调用时，不得以“保持流程一致”为理由加载它。

### Scarcity

用于表达真实的顺序约束：

- “实施前验证输入”；
- “修改后立即运行相应检查”。

不要虚构 deadline 或后果。

### Social proof

只用于说明经过验证的通用失败模式，例如：

> 未观察到 RED 的测试无法证明自己能捕获目标缺陷。

不要以“大家都这样做”为唯一理由。

### Unity

用协作语言鼓励真实反馈：

> 我们共同目标是得到可验证、可维护的结果；如果规则与项目现实冲突，请明确指出。

避免把协作变成迎合用户。

## 不建议使用

- **Reciprocity：**容易产生操控感，Skill 通常不需要。
- **Liking：**会增加 sycophancy，不适合 compliance。
- 虚假的 Authority、Scarcity 或 Social proof。

## 按失败类型使用

| Skill 类型 | 建议 |
|---|---|
| Discipline | 适度使用 Authority + Commitment |
| Technique | 清晰 recipe，少量 Authority |
| Collaborative | Unity + 明确选择 |
| Reference | 只追求清晰，不使用 persuasion |

## 伦理检查

应用前询问：

1. 这项规则是否已由用户授权？
2. 用户完全理解这种措辞后，它是否仍服务于用户利益？
3. 是否存在更简单的结构化 contract？
4. 是否会扩大工具、Git、网络或外部操作权限？

任一答案不清楚时，不要使用 persuasion 强化。

## 参考

- Cialdini, R. B. (2021), *Influence: The Psychology of Persuasion*.
- Meincke et al. (2025), *Call Me A Jerk: Persuading AI to Comply with Objectionable Requests*.
