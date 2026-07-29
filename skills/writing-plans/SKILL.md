---
name: writing-plans
description: 把已批准的 spec 或明确 requirements 转换成包含文件、步骤、测试、验证和提交边界的实施计划。仅当用户明确要求使用 writing-plans，或明确选择写有该名称的选项时使用；不要把否定、询问、文档提及或任务匹配视为调用授权。
---

# 编写实施计划

## 概述

把已批准的 spec 或明确 requirements 转换为全面、可逐项执行的实施计划。假定执行者熟悉开发，但不了解当前代码库、领域背景和本次讨论。

开始时说明正在使用 `writing-plans`。本技能只编写和自审计划，不实施计划，也不自动提交计划文档。

## 激活边界

只有用户肯定地要求使用 `writing-plans`，或明确选择写有该技能名称的选项时才能调用。否定使用、询问功能、打开 spec、requirements 或计划模板本身不构成授权。

## 输出位置

默认保存到：

```text
docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md
```

用户指定其他位置时，以用户指定为准。计划写完后保持未提交。

## 范围检查

开始编写前完整读取来源 spec 或 requirements。若输入明确标记为尚未批准的 draft spec，不把它擅自视为批准版本；询问用户是返回审阅，还是明确授权把当前内容当作 requirements 使用。

若输入覆盖多个独立子系统，建议分别创建计划。每份计划都应产生可以独立运行和验证的交付物。

## 先确定文件结构

定义任务前，列出将创建或修改的文件及其职责：

- 优先选择职责单一、边界清晰的文件。
- 会一起修改的内容按职责放在一起，而不是机械按技术层拆分。
- 遵循现有代码库模式。
- 只把服务当前目标的必要重构纳入计划。

该结构决定任务边界。每个顶层 `Task` 都应形成可独立理解、实现和验证的交付物。

## 任务粒度

顶层 `Task` 是值得单独验证和提交的最小交付单元。每个 Task 内的步骤应足够小，通常一次只做一个动作：

- 写一个失败测试；
- 运行并确认按预期失败；
- 写最小实现；
- 运行并确认通过；
- 提交该 Task。

不要把每个 2–5 分钟步骤误当成独立提交单元；默认每个顶层 Task 提交一次。

## 计划文档头部

每份计划以如下结构开始：

```markdown
# [功能名称] Implementation Plan

> **面向执行 Agent：**逐个 Task 执行此计划，并遵循其中规定的每项测试、
> 验证和 commit 步骤。除非用户明确要求，否则不要调用任何 Skill。
> 如果用户明确要求使用 `executing-spec-or-plan`，再用该 Skill 执行此计划。

**Goal：**[用一句话说明要构建什么]

**Architecture：**[用 2–3 句话说明实现方法]

**Tech Stack：**[关键技术与 libraries]

---
```

计划头部不能把任何其他技能标记为必需依赖，也不能把阅读计划视为调用技能的授权。

## Task 模板

````markdown
### Task N: [组件名称]

**Files:**
- Create: `准确/路径/file.py`
- Modify: `准确/路径/existing.py`
- Test: `tests/准确/路径/test.py`

**Interfaces:**
- Consumes: [前序 Tasks 提供的准确 signatures]
- Produces: [后续 Tasks 使用的准确名称、parameters 和 return types]

- [ ] **Step 1：编写失败测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2：运行测试并确认失败**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL，因为目标行为尚未实现

- [ ] **Step 3：编写最小实现**

```python
def function(input):
    return expected
```

- [ ] **Step 4：运行测试并确认通过**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5：提交当前 Task**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

计划中的 commit 步骤必须保留。计划执行时，用户可在执行技能的集中确认面板中明确覆盖提交策略。

## 不得使用占位符

每一步都要包含执行者实际需要的内容。以下属于计划失败：

- `TBD`、`TODO`、`implement later`；
- “添加适当的错误处理/验证/边界情况”，但没有具体规则；
- “为以上内容编写测试”，但没有具体测试；
- “与 Task N 类似”，要求执行者自行补全；
- 代码步骤只说做什么，却不给出如何做；
- 引用从未定义的 types、functions、methods 或 file paths；
- 缺少准确命令或预期结果。

## 自审

写完整份计划后，当前 Agent 自行检查，不派遣 reviewer：

1. **Spec 覆盖：**每项需求是否对应至少一个 Task？
2. **占位符：**是否存在模糊、延期或缺失的实现内容？
3. **接口一致性：**后续任务使用的名称和签名是否与前面定义一致？
4. **可构建性：**执行者是否能仅凭计划和代码库完成每一步？
5. **提交边界：**每个顶层 Task 是否在验证后有一个明确 commit？

发现问题时内联修复。自审不调用其他技能。

## 完成后的三个选项

计划写完并自审后，说明计划路径和未提交状态，然后严格提供：

1. **提交文档并结束**
   - 有来源 spec 文件时，只暂存并提交该 spec 与新计划。
   - 没有来源 spec 文件时，只提交新计划。
   - 不包含工作区其他修改。
   - 提交完成后结束，不实施计划。
2. **使用 `executing-spec-or-plan`**
   - 以刚生成的计划为输入进入执行前确认。
   - 计划和 spec 此时仍保持原有 Git 状态。
3. **直接结束**
   - 不提交，不实施。

必须等待用户明确选择三个选项之一。只有用户明确选择第一项才可以提交；只有用户明确选择第二项或直接点名 `executing-spec-or-plan` 才可以调用该技能；选择第三项才结束且不做其他操作。

### 无 spec 或无 Git

- 没有来源 spec 时，第一项只提交计划。
- 当前目录不是 Git 仓库时，说明无法执行第一项，但计划生成仍然成功。
- 任何提交都只包含本次生成的文档。
