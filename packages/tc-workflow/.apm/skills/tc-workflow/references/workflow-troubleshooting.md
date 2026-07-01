# workflow troubleshooting

读取本文件时，同时读取 `workflow-schema.md`。先定位失败类别，再给出最小修复。

## schema 不是 `1`

症状：validate 报告 schema 缺失或不是 `1`。

修复：

```yaml
schema: 1
```

## stage id 重复

症状：validate 报告 stage `id` 不唯一。

修复：为每个 stage 使用稳定且唯一的 id，例如 `design`、`plan`、`tasks`、`implement`、`review`、`unit-tests`。

## 缺少 `use`

症状：`kind: skill` 或 `kind: command` 阶段缺少 `use`。

修复：

```yaml
- id: review
  kind: skill
  use: tc-review
```

或：

```yaml
- id: plan
  kind: command
  use: tc.plan
```

## 缺少 `run`

症状：`kind: shell` 阶段缺少 `run`。

修复：加入用户明确要求或同意的命令。

```yaml
- id: unit-tests
  kind: shell
  run: pytest -q
```

## `requires` 的逻辑产物缺失

症状：运行到某阶段前停止，报告缺少 `spec.md`、`plan.md`、`tasks.md`、`code` 或 `tests`。

修复：

- 如果产物应该由前一阶段生成，给前一阶段补 `produces`。
- 如果当前 workflow 应从已有产物开始，确认当前 feature 路径和文件存在。
- 如果该阶段可跳过，增加 `optional: true` 并选择 `on_failure: ask` 或 `on_failure: continue`。

## on_failure: continue 未配合 optional: true

症状：validate 报告 `on_failure: continue` 使用非法。

修复：

```yaml
optional: true
on_failure: continue
```

如果阶段不是可选阶段，把 `on_failure` 改成 `stop` 或 `ask`。

## 当前 feature 路径无法解析

症状：workflow 需要当前 feature 产物，但路径解析失败。

修复：

- 如果 workflow 首阶段能创建 feature，例如 `tc-brainstorm`、`tc.specify` 或 `tc-quick`，从首阶段开始运行。
- 如果 workflow 从 `plan`、`tasks`、`implement`、`review` 等中间阶段开始，先切到已有 feature 上下文。
- 需要路径细节时运行项目里的 prerequisites 脚本，优先使用 `scripts/powershell/check-prerequisites.ps1 -PathsOnly -Json` 或 `scripts/bash/check-prerequisites.sh --paths-only --json`。
