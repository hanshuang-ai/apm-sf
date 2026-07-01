# workflow field guide

本文件帮助选择 workflow stage 字段。字段含义以 `workflow-schema.md` 为准；这里说明何时使用。

## mode

| mode | 使用场景 |
|------|----------|
| `manual` | 用户要自己执行该阶段，agent 只提示下一步 |
| `confirm` | 阶段会改代码、切上下文、迁移产物，执行前需要确认 |
| `auto` | 阶段依赖清楚、风险低，满足 `requires` 后可以自动执行 |

默认选择 `confirm`。只有常规串联阶段才使用 `auto`。

## gate.type

| gate.type | 使用场景 |
|-----------|----------|
| `none` | 阶段结束后无需停顿 |
| `user-approval` | 生成了需要用户审阅的产物，例如 `spec.md`、`tasks.md` 或迁移结果 |
| `pass` | 阶段本身是检查、评审或验证，失败时必须按 `on_failure` 处理 |

## requires 和 produces

`requires` 描述执行前必须已有的逻辑产物。`produces` 描述执行后预期出现的逻辑产物。

常见依赖：

| 阶段 | requires | produces |
|------|----------|----------|
| `tc-brainstorm` | 无 | `spec.md` |
| `tc.plan` | `spec.md` | `plan.md` |
| `tc.tasks` | `plan.md` | `tasks.md` |
| `tc.implement` | `tasks.md` | `code`, `tests` |
| `tc-review` | `code`, `tests` | 无固定逻辑产物 |

不要用 `requires` 表达希望执行的动作。缺少产物时，workflow 会停止或跳过 optional 阶段。

## optional 和 on_failure

| 组合 | 含义 |
|------|------|
| `optional: false` + `on_failure: stop` | 核心阶段失败即停止 |
| `optional: false` + `on_failure: ask` | 核心阶段失败后让用户决定重试、跳过或停止 |
| `optional: true` + `on_failure: ask` | 可选阶段失败后让用户决定 |
| `optional: true` + `on_failure: continue` | 可选阶段失败后记录并继续 |

`on_failure: continue` 必须配合 `optional: true`。非 optional 阶段不要使用 `continue`。

## always_run

默认情况下，如果 `produces` 已满足，stage 可以被跳过或询问跳过。需要强制执行检查、评审或刷新产物时，使用：

```yaml
always_run: true
```

常见用途：

- 每次都运行 review。
- 每次都运行 shell 测试。
- 即使 `plan.md` 已存在，也要重新生成计划。
