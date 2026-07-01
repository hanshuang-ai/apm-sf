---
name: tc-workflow
description: 需要按项目自定义流程串联多个 tc skill、speckit command、extension command、shell 脚本或人工 gate 时使用。读取 .specify/tc/workflows/*.yml，按阶段检查 requires/produces、处理 gate 和失败策略；只做编排，不重新实现阶段逻辑。
assets:
  - references/workflow-schema.md
  - references/workflow-examples.md
  - references/workflow-authoring-guide.md
  - references/workflow-field-guide.md
  - references/workflow-cookbook.md
  - references/workflow-troubleshooting.md
  - assets/brainstorm-quick.yml
  - assets/web-design-build.yml
  - assets/native-speckit-review.yml
  - assets/bugfix-artifact-repair.yml
  - assets/bugfix-native-implement.yml
  - assets/brownfield-onboard.yml
---

# TC Workflow

按项目自定义 workflow 串联任意阶段：tc skill、speckit command、extension command、shell 脚本、人工 gate 或说明节点。

## 用户输入

```text
$ARGUMENTS
```

参数格式：

```text
<workflow-name> [workflow arguments...]
```

示例：

```text
/tc-workflow web-design-build 增加首页活动配置页面
```

## 用户触发方式

支持以下入口：

```text
/tc-workflow list
/tc-workflow show <workflow-name>
/tc-workflow validate <workflow-name>
/tc-workflow run <workflow-name> [workflow arguments...]
/tc-workflow <workflow-name> [workflow arguments...]
```

`/tc-workflow <workflow-name> [workflow arguments...]` 是 `/tc-workflow run <workflow-name> [workflow arguments...]` 的简写。省略 `run` 时按运行 workflow 处理。

### `list`

列出项目 workflow 和内置 preset：

- 文件名
- `name`
- `description`
- stage 数量
- 来源：project / built-in

如果项目目录不存在或没有 workflow，仍列出内置 preset。若两者都没有，提示创建 `.specify/tc/workflows/<name>.yml`，并引用 `__ASSET_DIR__/references/workflow-examples.md`。

### `show <workflow-name>`

读取 workflow 文件并展示摘要：

- workflow 名称和描述
- stages 顺序
- 每个 stage 的 `id`、`kind`、`use` / `run`
- `requires` / `produces`
- `mode`、`gate`、`optional`、`on_failure`

不执行任何 stage。

### `validate <workflow-name>`

读取 workflow 文件并按 `__ASSET_DIR__/references/workflow-schema.md` 校验：

- schema 必须为 `1`
- stage id 不重复
- kind 合法
- `skill` / `command` stage 有 `use`
- `shell` stage 有 `run`
- `on_failure: continue` 只能与 `optional: true` 同用

只报告校验结果，不执行任何 stage。

### `run <workflow-name> [workflow arguments...]`

运行指定 workflow。`[workflow arguments...]` 写入 `${arguments}` 变量。

### 省略 `run`

当首个参数不是 `list`、`show`、`validate`、`run` 时，把它当作 workflow name：

```text
/tc-workflow brainstorm-quick 增加登录页验证码
```

等价于：

```text
/tc-workflow run brainstorm-quick 增加登录页验证码
```

## 职责边界

本技能只做编排：

- 读取 workflow YAML
- 校验阶段字段和产物依赖
- 判断断点续跑
- 按阶段调用指定 skill / command / shell / gate
- 在 gate 处暂停等待用户确认
- 根据 `on_failure` 处理失败

本技能绝不：

- 重新实现 `tc-brainstorm`、`tc-quick`、`tc-review` 等阶段逻辑
- 在阶段未声明的情况下自动补做隐藏步骤
- 绕过被调用 skill 自己的门禁、TDD 或自检要求
- 在用户拒绝 gate 后继续执行后续阶段

## 按需参考资料

不要一次性读取所有 workflow 参考文件。根据用户意图按需加载：

| 用户意图 | 读取资料 |
|----------|----------|
| 创建或修改 workflow | `__ASSET_DIR__/references/workflow-authoring-guide.md` |
| 判断 stage 字段如何选择 | `__ASSET_DIR__/references/workflow-field-guide.md` |
| 基于 preset 改造或需要示例 | `__ASSET_DIR__/references/workflow-examples.md` 和 `__ASSET_DIR__/references/workflow-cookbook.md` |
| validate/run 失败后排查 | `__ASSET_DIR__/references/workflow-schema.md` 和 `__ASSET_DIR__/references/workflow-troubleshooting.md` |
| 只 list/show/validate/run | 优先读取目标 workflow YAML；除非需要解释或修复，不读取教程类资料 |

当用户要求解释、创建、复制、修改或修复 workflow 时，先按上表读取最小必要资料，再继续处理。普通执行 workflow 时不要加载 cookbook 或 authoring guide。

## Workflow 文件位置

从项目根查找：

1. `.specify/tc/workflows/<workflow-name>.yml`
2. `.specify/tc/workflows/<workflow-name>.yaml`
3. `__ASSET_DIR__/assets/<workflow-name>.yml`（内置 preset）

项目 workflow 优先于内置 preset，同名时视为项目覆盖。找不到则停止，列出可用 workflow，并提示创建 `.specify/tc/workflows/<workflow-name>.yml`。可参考 `__ASSET_DIR__/references/workflow-examples.md`。

## 内置 presets

以下 workflow 随 `tc-workflow` 默认装配到 `__ASSET_DIR__/assets/`，可直接运行，也可复制到 `.specify/tc/workflows/` 后按项目修改：

| 文件 | 用途 |
|------|------|
| `brainstorm-quick.yml` | 深度设计并批准 spec.md 后，快速自动实现 |
| `web-design-build.yml` | 使用 Visual Companion 做网页设计，再快速实现、评审、E2E |
| `native-speckit-review.yml` | 原生 speckit specify → clarify → plan → tasks → implement 后追加评审 |
| `bugfix-artifact-repair.yml` | 记录 bug，修补 spec/plan/tasks，并验证规格产物一致性 |
| `bugfix-native-implement.yml` | 进阶 bugfix：切换上下文、修补产物、验证、实现、评审 |
| `brownfield-onboard.yml` | 既有项目接入：扫描、bootstrap、迁移、校验 |

## Schema

读取并遵循 `__ASSET_DIR__/references/workflow-schema.md`。第一版只支持顺序执行，不支持分支、循环、矩阵或并行。

必需字段：

- `schema`
- `name`
- `stages`
- 每个 stage 的 `id`、`kind`

## 变量

支持以下变量替换：

| 变量 | 含义 |
|------|------|
| `${arguments}` | workflow name 之后的全部用户输入 |
| `${workflow}` | workflow 名称 |
| `${stage}` | 当前 stage id |

不解释任意表达式；不要执行 YAML 中未定义的动态代码。

## 产物解析

逻辑产物按当前 feature 解析：

| 产物 | 判定 |
|------|------|
| `spec.md` | 当前 `FEATURE_SPEC` 存在 |
| `plan.md` | 当前 `IMPL_PLAN` 存在 |
| `tasks.md` | 当前 `TASKS` 存在 |
| `bug-report` | 当前 feature 的 `bugs/BUG-NNN.md` |
| `brownfield-report` | brownfield scan 输出的 Project Profile / 项目扫描报告 |
| `code` | 不能仅靠文件判断；由阶段报告或后续检查确认 |
| `tests` | 不能仅靠文件判断；由测试命令或阶段报告确认 |

路径解析优先运行：

- sh：`scripts/bash/check-prerequisites.sh --paths-only --json`
- ps：`scripts/powershell/check-prerequisites.ps1 -PathsOnly -Json`

如果当前没有 feature 且首个阶段能创建 feature（如 `tc-brainstorm`、`tc.specify`、`tc-quick`），允许从首阶段开始；否则停止并提示先创建 feature。

## 执行流程

1. 解析 `$ARGUMENTS`，取得 workflow name 和 workflow arguments。
2. 读取 workflow YAML。
3. 校验 schema 和 stage 列表：
   - `id` 不重复
   - `kind` 属于 `skill | command | shell | gate | note`
   - 非 `gate` / `note` stage 必须有 `use`
   - `on_failure` 属于 `stop | ask | continue`
4. 解析当前 feature 路径和已有产物。
5. 逐个 stage 执行：
   - `requires` 缺失且 `optional: true` → 跳过并记录
   - `requires` 缺失且非 optional → 停止并报告缺失产物
   - `produces` 已满足且 stage 未声明 `always_run: true` → 询问是否跳过；若 `mode: auto` 可自动跳过
   - 执行 stage
   - 检查 `produces`
   - 处理 `gate`
6. 结束时报告执行过、跳过、失败的阶段，以及下一步建议。

## Stage 行为

### `kind: skill`

调用 `use` 指定的 skill。示例：

```yaml
- id: design
  kind: skill
  use: tc-brainstorm
  args: "${arguments}"
```

执行前必须读取目标 skill 的说明，并遵守其门禁。

### `kind: command`

调用 speckit 或 extension 命令。示例：

```yaml
- id: plan
  kind: command
  use: tc.plan
```

命令名只作为 agent 指令，不自行猜测内部实现。

### `kind: shell`

仅执行 workflow 明确写出的 `run` 命令。示例：

```yaml
- id: unit-tests
  kind: shell
  run: pytest tests/test_skills.py -q
```

执行前展示命令。失败按 `on_failure` 处理。

### `kind: gate`

人工确认点。示例：

```yaml
- id: approve-spec
  kind: gate
  gate:
    type: user-approval
```

必须停下来等待用户回复。

### `kind: note`

只输出说明，不执行动作。

## Gate

支持：

| gate.type | 行为 |
|-----------|------|
| `none` | 不停顿 |
| `user-approval` | 展示阶段产物并等待批准 |
| `pass` | 要求前置检查通过；失败则按 `on_failure` |

未声明 gate 时默认 `none`。

## 失败策略

| `on_failure` | 行为 |
|--------------|------|
| `stop` | 停止 workflow，报告失败阶段 |
| `ask` | 询问用户重试、跳过或停止 |
| `continue` | 记录失败并继续，仅允许 `optional: true` stage 使用 |

默认 `stop`。

## 完成报告

报告：

- workflow 名称
- 执行阶段
- 跳过阶段和原因
- 失败阶段和原因
- 关键产物路径
- 建议下一步
