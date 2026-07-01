# workflow schema

## 文件位置

```text
.specify/tc/workflows/<workflow-name>.yml
.specify/tc/workflows/<workflow-name>.yaml
```

## 最小结构

```yaml
schema: 1
name: brainstorm-quick
description: 深度设计后快速自动实现

stages:
  - id: design
    kind: skill
    use: tc-brainstorm
    args: "${arguments}"
    produces:
      - spec.md
    gate:
      type: user-approval

  - id: implement
    kind: skill
    use: tc-quick
    requires:
      - spec.md
    mode: auto
    produces:
      - plan.md
      - code
      - tests
    on_failure: stop
```

## 顶层字段

| 字段 | 必需 | 含义 |
|------|------|------|
| `schema` | 是 | 当前固定为 `1` |
| `name` | 是 | workflow 名称，应与文件名一致 |
| `description` | 否 | 人类可读说明 |
| `inputs` | 否 | 预留；第一版只支持 `${arguments}` |
| `stages` | 是 | 顺序阶段列表 |

## Stage 字段

| 字段 | 必需 | 默认 | 含义 |
|------|------|------|------|
| `id` | 是 | 无 | 阶段唯一标识 |
| `kind` | 是 | 无 | `skill` / `command` / `shell` / `gate` / `note` |
| `use` | 条件 | 无 | `skill` 或 `command` 的目标名称 |
| `run` | 条件 | 无 | `shell` 阶段执行的命令 |
| `args` | 否 | 空 | 传给目标的文本参数 |
| `with` | 否 | `{}` | 结构化配置，供目标 skill 理解 |
| `requires` | 否 | `[]` | 执行前必须存在的逻辑产物 |
| `produces` | 否 | `[]` | 执行后期望存在的逻辑产物 |
| `mode` | 否 | `confirm` | `manual` / `confirm` / `auto` |
| `gate` | 否 | `none` | 阶段后门禁 |
| `optional` | 否 | `false` | 是否允许跳过 |
| `on_failure` | 否 | `stop` | `stop` / `ask` / `continue` |
| `always_run` | 否 | `false` | 即使产物已存在也执行 |

`kind: gate` 和 `kind: note` 不需要 `use`。

## kind

| kind | 用途 |
|------|------|
| `skill` | 调用 tc skill 或用户自定义 skill |
| `command` | 调用 speckit core command 或 extension command |
| `shell` | 执行显式 shell 命令 |
| `gate` | 人工确认点 |
| `note` | 输出说明，不执行动作 |

## mode

| mode | 行为 |
|------|------|
| `manual` | 只提示用户该执行什么，不自动调用 |
| `confirm` | 执行前确认 |
| `auto` | 满足条件时自动执行 |

## gate

```yaml
gate:
  type: none | user-approval | pass
```

| type | 行为 |
|------|------|
| `none` | 不停顿 |
| `user-approval` | 展示产物，等待用户批准 |
| `pass` | 要求检查通过，否则按 `on_failure` |

## 逻辑产物

| 产物 | 含义 |
|------|------|
| `spec.md` | 当前 feature 的规格 |
| `plan.md` | 当前 feature 的实现计划 |
| `tasks.md` | 当前 feature 的任务列表 |
| `research.md` | 当前 feature 的研究记录 |
| `data-model.md` | 当前 feature 的数据模型 |
| `contracts/` | 当前 feature 的契约目录 |
| `quickstart.md` | 当前 feature 的验证说明 |
| `bug-report` | 当前 feature 的 `specs/<feature>/bugs/BUG-NNN.md` |
| `brownfield-report` | brownfield scan 输出的 Project Profile / 项目扫描报告 |
| `code` | 代码改动，由阶段报告或检查命令确认 |
| `tests` | 测试改动或测试结果，由阶段报告或检查命令确认 |

## 变量

| 变量 | 含义 |
|------|------|
| `${arguments}` | workflow name 后的全部输入 |
| `${workflow}` | workflow 名称 |
| `${stage}` | 当前 stage id |

第一版不支持表达式、条件语法、循环、矩阵或并行。

## 校验规则

- `schema` 必须为 `1`
- `stages` 必须为非空列表
- stage `id` 必须唯一
- stage `kind` 必须属于支持列表
- `skill` / `command` stage 必须有 `use`
- `shell` stage 必须有 `run`
- `on_failure: continue` 只能与 `optional: true` 同用
- `gate.type` 未声明时按 `none` 处理
- 不允许 workflow 隐式执行未声明的阶段
