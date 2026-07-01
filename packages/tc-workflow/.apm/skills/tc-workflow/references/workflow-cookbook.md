# workflow cookbook

本文件提供小型改造片段。使用前先读取目标 preset 或项目 workflow，只做与用户目标相关的最小改动。

## 从 `brainstorm-quick` 改成只生成 `spec.md`

删除 `implement` 阶段，只保留设计阶段：

```yaml
stages:
  - id: design
    kind: skill
    use: tc-brainstorm
    args: "${arguments}"
    produces:
      - spec.md
    gate:
      type: user-approval
```

适合用户只想先完成需求澄清和规格批准。

## 从 `web-design-build` 去掉 E2E

删除 `e2e` 阶段，保留 design、implement、review：

```yaml
  - id: review
    kind: skill
    use: tc-review
    requires:
      - code
      - tests
    gate:
      type: pass
    on_failure: ask
```

适合项目没有 E2E 环境，或本次只要求实现后评审。

## 已有 `spec.md`，从 `tc.plan` 开始

workflow 的第一阶段可以直接要求 `spec.md`：

```yaml
stages:
  - id: plan
    kind: command
    use: tc.plan
    requires:
      - spec.md
    produces:
      - plan.md

  - id: tasks
    kind: command
    use: tc.tasks
    requires:
      - plan.md
    produces:
      - tasks.md
    gate:
      type: user-approval
```

适合用户已经有当前 feature 的规格文件。

## 增加 shell 测试阶段

只写用户明确要求或同意的命令：

```yaml
  - id: unit-tests
    kind: shell
    run: pytest -q
    requires:
      - code
    gate:
      type: pass
    on_failure: stop
    always_run: true
```

如果测试环境不稳定，可以把 `on_failure` 改为 `ask`，不要把核心测试直接设为 `continue`。

## 增加人工审批 gate

需要用户审阅产物时，在阶段后增加：

```yaml
gate:
  type: user-approval
```

常见位置：

- `tc-brainstorm` 生成 `spec.md` 后。
- `tc.tasks` 生成 `tasks.md` 后。
- brownfield bootstrap 或 migrate 后。
