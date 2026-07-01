# 实现计划：[FEATURE NAME]

**分支**：[BRANCH] | **Spec**：[链接到 spec.md]

## Technical Context

- 语言/版本：[填写，未知标 NEEDS CLARIFICATION]
- 主要依赖/框架：[填写]
- 存储/数据：[填写或 N/A]
- 测试框架：[填写]
- 目标平台：[填写]
- 项目结构：[单体 / web 前后端 / CLI 等，给出关键目录]
- 约束：[性能/兼容/安全等，无则 N/A]

## Constitution Check

> 依据 `.specify/memory/constitution.md` 逐条核对；存在违反且无正当理由 → 停止并报错。

- [ ] [原则1 名称]：[本设计如何满足 / 偏离理由]
- [ ] [原则2 名称]：[同上]

## 文件结构

### 已有文件（修改）

| 文件路径 | 修改原因 | 关联 FR-ID / US# |
|---------|---------|-----------------|
| [path] | [reason] | [FR-xxx / US#] |

### 新增文件（创建）

| 文件路径 | 创建原因 | 关联 FR-ID / US# |
|---------|---------|-----------------|
| [path] | [reason] | [FR-xxx / US#] |

### FR-ID → 文件映射

| FR-ID | 涉及文件 |
|-------|---------|
| FR-001 | [file1, file2] |

## 方案说明

[基于 spec.md Assumptions 中选择的方案，简要说明实现策略。2-3 句话描述技术路线。]

## Tasks

> 严格 checklist 格式：`- [ ] T001 [P] [US1] 描述 + 文件路径`
> [P] 仅用于可并行（不同文件、无未决依赖）；用户故事阶段任务必须带 [US#] 标签。
> 每个实现任务内嵌 TDD 步骤（RED → GREEN → REFACTOR → Commit），与 /tc-tdd 对齐。
> 以下写法是 **plan failure**：TBD/TODO、"add appropriate error handling"、"implement validation"、"write tests"、任何无法在 2-5 分钟内完成的步骤描述。

### Phase 1: Setup

- [ ] T001 [描述 + 文件路径]
  1. RED: 在 tests/path/test_name.ext 写失败测试，验证 [具体行为]
  2. Verify RED: 运行测试，确认因功能缺失而失败
  3. GREEN: 在 src/path/file.ext 写最小实现代码通过测试
  4. Verify GREEN: 运行测试，确认通过且其他测试不受影响
  5. REFACTOR: 清理代码，保持测试通过
  6. Commit: git commit -m "T001: [简述]"

### Phase 2: Foundational（阻塞所有用户故事的前置）

- [ ] T00X [描述 + 文件路径]
  1. RED: ...
  2. Verify RED: ...
  3. GREEN: ...
  4. Verify GREEN: ...
  5. REFACTOR: ...
  6. Commit: ...

### Phase 3+: 用户故事（按 spec.md 优先级 P1、P2… 各成一阶段）

#### US1 - [User Story 标题]

- [ ] T0XX [US1] [描述 + 文件路径]
  1. RED: ...
  2. Verify RED: ...
  3. GREEN: ...
  4. Verify GREEN: ...
  5. REFACTOR: ...
  6. Commit: ...

#### US2 - [User Story 标题]

- [ ] T0XX [US2] [描述 + 文件路径]
  ...

### Final Phase: Polish & Cross-Cutting

- [ ] T0XX [描述 + 文件路径]
  ...
