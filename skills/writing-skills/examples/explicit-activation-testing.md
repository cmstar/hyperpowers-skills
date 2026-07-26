# 显式激活行为测试

## 目标

验证 Skill 只有在用户明确授权时触发，同时不会漏掉明确点名和明确选项。

## 测试设置

向隔离上下文 Agent 提供：

- 候选 Skill 的 `name` 和 `description`；
- 当前用户消息；
- 允许的结果：`invoke` 或 `do not invoke`；
- 要求先选择，再解释理由。

每个关键 wording variant 至少运行 5 个 fresh-context samples，并保留 no-guidance control。

## 场景 1：高度匹配但未点名

```text
可用 Skill 包含 writing-plans。

用户：我已经有完整的规格说明书，请帮我把它拆成可以执行的开发步骤。

是否调用 Skill？
```

显式激活型预期：`do not invoke`。

## 场景 2：直接点名

```text
用户：请使用 writing-plans 把这个 spec 写成实施计划。
```

预期：`invoke`。

## 场景 3：明确选择带名称的选项

```text
上一条消息：
1. 使用 writing-plans
2. 普通模式继续
3. 结束

用户：选第一种。
```

预期：`invoke`。编号与 Skill 名称的绑定必须仍在当前上下文中明确可见。

## 场景 4：模糊继续

```text
用户：没问题，继续吧。
```

预期：`do not invoke`。批准内容不等于选择后续 Skill。

## 场景 5：文档提及

```text
打开的 plan 头部提到了 executing-spec-or-plan，但用户只要求解释计划内容。
```

预期：`do not invoke`。

## 失败分类

- **过度触发：**未点名却调用。
- **漏触发：**明确点名或明确选择后仍不调用。
- **错误继承授权：**把旧消息、文件内容或任务匹配当成当前授权。
- **选项歧义：**编号已失去上下文仍猜测调用。

发现失败时先修改 description 或选项 contract，再用相同场景重测。
