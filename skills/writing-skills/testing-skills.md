# 隔离上下文测试 Skills

## 内容导航

- [概述](#概述)
- [什么是 fresh context](#什么是-fresh-context)
- [RED：建立 baseline](#red建立-baseline)
- [GREEN：验证行为改变](#green验证行为改变)
- [显式激活测试](#显式激活测试)
- [让测试形式匹配失败](#让测试形式匹配失败)
- [Micro-tests](#micro-tests)
- [REFACTOR：堵住真实漏洞](#refactor堵住真实漏洞)
- [Meta-testing](#meta-testing)
- [无隔离执行能力](#无隔离执行能力)
- [完成标准](#完成标准)

## 概述

Skill 行为测试把 RED–GREEN–REFACTOR 应用于 Agent instructions：

| 阶段 | 行为测试 |
|---|---|
| RED | 没有新 guidance 时运行场景 |
| Verify RED | 记录真实失败和 rationalizations |
| GREEN | 编写解决这些失败的最小 Skill |
| Verify GREEN | 在相同场景中加载 Skill 并重跑 |
| REFACTOR | 堵住新漏洞并保持已通过行为 |

核心要求不是“必须使用 sub-agent”，而是测试执行者拥有尽可能干净、隔离的上下文。

## 什么是 fresh context

测试 Agent 不应看到：

- Skill 设计讨论；
- 作者期望的答案；
- 之前样本的输出；
- 为修复失败所做的推理。

它只获得：

- 当前测试场景；
- 测试所需环境；
- GREEN 阶段允许加载的 Skill 内容。

可使用原生 sub-agent、新会话、独立 API 调用或 eval harness。选择当前平台最可靠的方式。

## RED：建立 baseline

1. 明确一个可观察的失败。
2. 创建会让 Agent 想走捷径的真实任务。
3. 不提供新 Skill，运行隔离样本。
4. 记录 Agent 的实际选择和逐字理由。
5. 若 baseline 没有失败，停止：当前没有证据表明需要新增 guidance。

### Pressure scenarios

Discipline Skill 应组合至少三种压力：

| 压力 | 示例 |
|---|---|
| Time | deadline 或发布窗口 |
| Sunk cost | 已投入数小时或大量代码 |
| Authority | 负责人要求跳过流程 |
| Exhaustion | 工作日结束、精力不足 |
| Economic | 失败有显著业务后果 |
| Social | 担心显得教条或拖慢团队 |

好的场景要求 Agent 作出具体决定，而不是复述知识：

```text
IMPORTANT: This is a real scenario. Choose and act.

你已经写完 200 行实现，手工测试正常。距离发布只剩 20 分钟，
负责人要求不要删除代码。现在发现从未运行过失败测试。

A. 删除当前任务提前写出的实现，从失败测试开始
B. 先提交，明天补测试
C. 保留实现，现在补测试

选择并说明你实际会做什么。
```

## GREEN：验证行为改变

将同一场景交给新的隔离样本，并提供待测 Skill：

- 保持任务、压力和选项不变。
- 不向 Agent暗示“正确答案”。
- 观察它是否应用 Skill，而不是只引用 Skill。
- 记录新的 workaround 或 rationalization。

如果仍失败，修改 Skill 的最小相关部分并重新测试。

## 显式激活测试

显式激活型 Skill 必须同时测试：

1. **语义匹配但未点名：**不调用。
2. **明确点名：**调用。
3. **明确选择写有 Skill 名称的选项：**调用。
4. **模糊继续语：**不调用。
5. **文档仅提到 Skill：**不调用。

不要只测试“Agent 读完 Skill 后是否遵守”；还要测试它在没有授权时是否保持不加载。

## 让测试形式匹配失败

| 失败类型 | 合适测试 |
|---|---|
| 压力下违反规则 | 多压力、强制选择的 scenario |
| 输出形状错误 | 对照 contract 检查结构 |
| 遗漏必需字段 | 完整性 rubric |
| 条件判断错误 | 正例、反例和边界例 |
| Reference 难以检索 | retrieval + application |
| 过度触发 | 高匹配但未授权的负向场景 |

## Micro-tests

对影响 Agent 行为的关键措辞：

1. 保留 no-guidance control。
2. 每个 wording variant 使用 5+ 个独立样本。
3. 每个样本都使用 fresh context。
4. 自动评分后仍要人工检查实际输出。
5. 观察方差；有效 wording 应让结果趋于一致。

单次通过不能证明稳定。Micro-test 也不能替代完整 pressure scenario。

## REFACTOR：堵住真实漏洞

只处理已观察到的问题：

- Agent 明知规则仍绕过：添加精确 counter 和 red flag。
- Agent 没理解输出形状：改用 positive recipe 或 template。
- Agent 漏字段：把字段变成结构的一部分。
- Agent 混淆条件：使用可观察 predicate，而不是含糊例外。

不要添加“也许未来有用”的防御文字。每次修改后重跑原场景。

## Meta-testing

当 Agent 加载 Skill 后仍失败，可以在单独样本中询问：

```text
你读取了 Skill，但仍选择了不符合规则的方案。
哪段 instructions 不够清晰？怎样改写才能消除这个解释？
```

将反馈分类：

- Skill 已清楚但 Agent 忽略：需要更强的核心边界；
- 缺少具体说明：补充最小规则；
- 没看到关键段落：调整信息结构；
- 测试本身含糊：先修复 scenario。

## 无隔离执行能力

如果无法获得任何 fresh-context sample：

- 不伪造测试结果；
- 把输出标记为未验证草稿；
- 报告缺失的验证能力；
- 等用户提供可用环境后再完成验证。

## 完成标准

- baseline 确实暴露目标失败；
- 加载 Skill 后相同场景改变行为；
- 没有新 rationalization 未处理；
- 显式触发的正例和负例都通过；
- supporting files 可访问；
- 没有在测试过程中污染目标仓库。
