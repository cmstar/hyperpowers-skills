# Agent Skills 平台无关最佳实践

## 内容导航

- [核心原则](#核心原则)
- [可移植结构](#可移植结构)
- [Frontmatter](#frontmatter)
- [Progressive disclosure](#progressive-disclosure)
- [Workflow 与反馈回路](#workflow-与反馈回路)
- [Scripts 与依赖](#scripts-与依赖)
- [Examples](#examples)
- [Testing](#testing)
- [发布前检查](#发布前检查)

## 核心原则

- 假定 Agent 已掌握通用知识，只补充真正缺失的 context。
- 让具体程度匹配任务脆弱性：开放任务给 heuristics，危险操作给精确 guardrails。
- 用真实使用和隔离上下文样本验证 Skill。
- 将 activation policy 当作公开契约，而不是隐含行为。

## 可移植结构

```text
skill-name/
├── SKILL.md
├── reference.md
├── examples/
│   └── focused-example.md
└── scripts/
    └── deterministic-tool
```

- `SKILL.md` 是入口和导航。
- supporting files 与 SKILL.md 保持一层引用深度。
- 使用 forward slashes。
- 不写依赖某台机器的绝对路径。
- 不假定某个 runtime 一定提供某项交互工具、shell、sub-agent、MCP 或 package manager。
- 需要特定能力时写出检测方式和降级路径。

## Frontmatter

最低字段是 `name` 和 `description`。字段限制以目标 runtime 和 [Agent Skills specification](https://agentskills.io/specification) 为准。

显式激活型 description 应同时表达：

- 哪种用户表达构成授权；
- 哪些高匹配场景仍不得自动调用。

自动发现型 description 只有在用户明确要求时使用，并应测试过度触发。

## Progressive disclosure

保持主文件可快速扫描：

- Overview 与核心边界放主文件。
- 大型 API/reference 放单独文件。
- 只有 Agent 需要执行的确定性工具才放 scripts。
- 每个 supporting file 都应有清晰的加载条件。
- 删除从未在真实测试中读取或使用的附件。

## Workflow 与反馈回路

复杂任务使用清晰阶段和完成条件：

```text
inspect → decide → execute → verify → report
```

需要反复修正时写出 feedback loop：

```text
run validator → inspect failure → fix → rerun
```

不要为线性步骤绘制复杂流程图。只有分支、循环或状态关系难以用短文本说明时才可视化。

## Scripts 与依赖

脚本适用于：

- 重复且确定性的操作；
- 容易手写出错的格式转换；
- 可机器验证的检查。

脚本必须：

- 有明确输入、输出和错误信息；
- 不包含无法解释的 magic values；
- 说明依赖；
- 在依赖不存在时给出安全降级；
- 不把本可在 instructions 中简单表达的判断过度工程化。

不强制使用任何单一 runtime 或交互工具。

## Examples

- 一个完整、真实、可改编的 example 胜过多个占位模板。
- Example 应展示关键 pattern，而不是重复解释常识。
- 输出格式严格时给 exact contract；允许判断时说明可调整边界。

## Testing

至少验证：

- activation 正例；
- activation 负例；
- 主 workflow；
- 一个边界或失败场景；
- supporting files 的可访问性；
- 目标 runtime 缺少可选能力时的降级。

对 behavior-shaping wording 使用 control 与多次独立样本。详细方法见 [testing-skills.md](testing-skills.md)。

## 发布前检查

- Skill 自包含；
- 无断开的相对链接；
- 无未声明的工具或 package 依赖；
- 无平台专属强制路径；
- 无自动调用其他 Skill；
- 无未经用户明确授权的 commit、push、PR 或外部写操作；
- 无 time-sensitive 断言，或已明确标注版本范围；
- 已报告真实验证状态。
