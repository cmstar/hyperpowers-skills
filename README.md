# Hyperpowers

Hyperpowers 是一个以 [Superpowers](https://github.com/obra/superpowers) 精简重构为基础的 Agent Skills 项目。它保留从需求设计到计划、执行、测试和 Skill 编写的核心能力，同时重新组织技能之间的关系，让每个阶段都可以单独使用。

本项目不是对原版工作流的替代，也不试图覆盖原版的全部能力。它面向希望保留核心工程方法、但更希望自己决定何时进入下一阶段的用户。

## 安装

确保本机可以使用 `npx`，然后运行：

```bash
npx skills add cmstar/hyperpowers-skills
```

按照交互提示选择需要安装的 Skills 和目标 Agent。

### 安装前必须卸载原版

Hyperpowers 与原版 Superpowers 不能同时安装。两者包含多个同名 Skill，同时存在时可能发生文件覆盖、重复发现或无法确定加载哪一个版本的问题。

安装本项目之前，必须先按照原版的安装方式卸载 Superpowers，并确认目标 Agent 的 Skill 目录中不再残留原版的同名 Skills。不要通过覆盖安装的方式混用两个版本。

### 可以从精简版切换回原版

安装不能共存，但开发过程可以迁移。如果使用 Hyperpowers 开发到一半后决定切换回原版，可以先卸载本项目，再安装 Superpowers，然后继续使用已有的：

- `docs/superpowers/specs/` 下的规格说明书；
- `docs/superpowers/plans/` 下的实施计划；
- 当前代码、测试和 Git 状态；
- plan 中的 Tasks、验证步骤和 commit 边界。

本项目保留了这些默认路径和核心文档结构，就是为了让中途切换不会丢失已经完成的设计与计划。

两套工作流的自动化程度和后续步骤并不完全相同。如果当前正在使用 `executing-spec-or-plan` 直接执行 spec、还没有正式 plan，切换回原版后可能需要先根据现有 spec 生成实施计划，再按照原版流程继续。

## 核心技能

| Skill | 核心功能 |
|---|---|
| `brainstorming` | 通过澄清问题、方案比较和用户审阅，把模糊想法整理成已批准的规格说明书（spec）。 |
| `writing-plans` | 把已批准的 spec 或明确 requirements 转换成包含文件、步骤、测试、验证和提交边界的实施计划。 |
| `executing-spec-or-plan` | 执行完整 plan，或跳过正式计划文档直接执行已批准 spec；实施前集中确认执行策略，解释任务拆解与 TDD 的建议依据，并在平台支持时确认 sub-agent 模型与推理强度。 |
| `test-driven-development` | 使用 RED–GREEN–REFACTOR 循环执行代码变更，并保护 legacy code 与用户已有修改。 |
| `writing-skills` | 使用 fresh-context 行为测试创建、更新和验证 Agent Skills。 |

## 辅助技能

| Skill | 核心功能 |
|---|---|
| `git-auto-commit` | 根据当前提交范围、项目规则和近期历史生成提交信息，并创建一次 Git commit。 |

`git-auto-commit` 是独立的显式调用技能，可以直接使用；`executing-spec-or-plan` 只有在用户选择明确写有该名称的提交策略后，才会在已授权的提交边界调用它。

## 基本思路

所有 Skills 都只接受肯定、明确的调用。以下情况不会触发 Skill：

- 任务内容与 Skill 高度匹配；
- 用户只说“继续”“开始”；
- plan、spec 或其他文档提到了 Skill；
- 用户只是在询问 Skill 的作用；
- 用户明确表示不要使用某个 Skill。

一个典型流程是：

1. 明确要求使用 `brainstorming`，形成并批准 spec。
2. 在 spec 批准后选择：
   - 使用 `writing-plans` 编写详细实施计划；
   - 使用 `executing-spec-or-plan` 直接执行 spec；
   - 到此结束。
3. 如果生成了 plan，可选择提交文档、执行 plan，或保持未提交并结束。
4. 执行前，由用户决定执行位置、提交策略、TDD 模式以及是否使用 sub-agent；若平台支持为 sub-agent 单独指定模型或推理强度，还需分别选择本次执行统一使用的选项。选择带有 `git-auto-commit` 名称的提交策略即授权按相应边界提交。

模型与推理强度的首项都叫“默认”：派遣时优先遵守 `AGENTS.md`、平台配置等现有规则；没有额外指定时才与当前会话相同。
5. 实施完成后执行最终验证，并报告 commits、未提交文件、branch 和 worktree 状态。

`writing-skills` 是一项独立的元技能，用于维护 Skill 本身，不属于每次开发任务都必须经过的主流程。

## 基本用法

直接在对话中明确说出要使用的 Skill，例如：

> 使用 `brainstorming` 帮我整理这个功能。

> 使用 `writing-plans` 把这份 spec 写成实施计划。

> /executing-spec-or-plan 执行这份计划。

> 编码过程使用 `test-driven-development`。

> /writing-skills 创建一个新的 Skill ，功能是 …… 。

> 使用 `git-auto-commit` 提交当前任务的改动。

如果没有明确点名，Agent 应按普通模式处理任务，不加载本项目定义的 Skill 工作流。

## 为什么精简

随着 coding Agent 使用的模型越来越聪明，原版为弥补早期模型不稳定而设计的大量固定步骤和自动衔接，在部分任务中可能占用过多上下文，并限制 Agent 根据代码库和任务规模自主判断。Hyperpowers 不取消设计、计划、TDD 和验证，而是把它们改为由用户按需显式启用：用户决定关键边界，Agent 在边界内自由选择实现方式，从而在保留工程纪律的同时减少流程开销，提高执行效率和结果质量。精简也不意味着只能删减原版内容；当一项独立能力符合这些原则时，项目可以把它作为扩展加入。

Hyperpowers 采用以下原则：

- **显式调用**：任务内容即使高度匹配，也不会自动触发 Skill；用户必须明确要求使用。
- **技能解耦**：一个 Skill 可以建议另一个 Skill，但不能强制调用。
- **保留退出路径**：设计、计划和执行阶段都允许用户在明确节点结束流程。
- **提交可控**：文档生成和代码执行不会把“完成”自动等同于 Git commit；执行器只在用户明确选择 `git-auto-commit` 提交策略后提交。
- **执行前集中确认**：在实施 plan 或 spec 前，一次性收集执行位置、提交、TDD、任务拆解和 sub-agent 等决策；仅在平台支持时收集 sub-agent 模型与推理强度。
- **保留核心纪律**：详细计划、RED–GREEN–REFACTOR、验证和安全边界仍然保留。

## 深入文档

完整的设计差异、技能调整说明、决策维度和工作流程见 [项目 Wiki](docs/wiki.md)。
