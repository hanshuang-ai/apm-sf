---
name: tc-writing-plans
description: 需要为已批准的 spec.md 生成详细实现计划时使用。先做范围检查和文件结构映射，再逐任务拆解（含 TDD 步骤模式），产出合并的 plan.md（整合计划 + 任务步骤）。是 /speckit.plan + /speckit.tasks 的增强替代选项；TDD 步骤模式与 /tc-tdd 对齐。
assets:
  - template.md
  - references/plan-self-review.md
  - references/file-structure-map.md
---

# Writing Plans

为已批准的 spec.md 生成详细实现计划，产出一份合并的 plan.md（整合计划 + 任务步骤）。

**开宗明义**："我正在使用 writing-plans 技能创建实现计划。"

## 用户输入

```text
$ARGUMENTS
```

如非空，作为补充上下文纳入考量。

## 前置：before_plan hook 检查

检查项目根的 `.specify/extensions.yml`：
- 不存在或无 `hooks.before_plan` 条目 → 跳过本节。
- 解析失败 → 静默跳过。
- 过滤 `enabled: false` 的 hook（无 `enabled` 字段视为启用）。
- 对每个可执行 hook，按 `optional` 输出 Optional Pre-Hook（提示）或 Automatic Pre-Hook（`EXECUTE_COMMAND: {command}`，等待结果后再继续）。

## Phase 0: 前置检查 + 范围校验

1. **路径解析**：从仓库根运行
   - sh：`scripts/bash/check-prerequisites.sh --paths-only --json`
   - ps：`scripts/powershell/check-prerequisites.ps1 -PathsOnly -Json`
   解析出 `FEATURE_DIR`、`FEATURE_SPEC`、`IMPL_PLAN` 等路径。

2. **加载 spec.md**：必须存在（来自 tc-brainstorm 或 /speckit.specify）。缺失则报错："请先运行 /tc-brainstorm 或 /speckit.specify 生成 spec.md"。

3. **加载 constitution**：读取 `.specify/memory/constitution.md`。

4. **加载 brief references**（如有）：读取 `docs/project/briefs/<feature-id>-references.md` 和 `docs/project/briefs/<feature-id>-handoff-notes.md`，作为 Technical Context 种子数据。

5. **范围检查**：如果 spec 覆盖多个独立子系统，询问拆分：
   - "本 spec 覆盖多个独立子系统。是否：(A) 保持单一 plan.md，按子系统分段 / (B) 拆分为多个 plan.md，每个子系统一个 feature / (C) 其他(请说明)"
   - 如果选 B：建议为每个子系统分别运行 tc-writing-plans。

6. **Constitution Gate**：检查 spec 是否违反 constitution 原则。违规且无正当理由 → 报错并停止。

## Phase 1: 文件结构映射

参考 `__ASSET_DIR__/references/file-structure-map.md`。

在任务拆解之前，先映射目标文件结构：

1. 扫描现有代码库结构
2. 读取 spec 的 Assumptions 章节了解架构方向
3. 产出文件结构映射：
   - 已有文件（修改）：文件路径、修改原因、关联 FR-ID / US#
   - 新增文件（创建）：文件路径、创建原因、关联 FR-ID / US#
   - 新增目录
   - FR-ID → 文件映射
   - US# → 文件映射

### 文件结构设计原则

- 每个文件单一清晰职责，通过定义良好的接口通信
- 偏好小而聚焦的文件，而非做太多的大文件
- 一起变化的文件放在一起，按职责分拆而非按技术层分拆
- 在现有代码库中遵循已有模式；如果修改的文件已经过于庞大，在计划中包含拆分是合理的

## Phase 2: 生成 plan.md

参考 `__ASSET_DIR__/template.md`。

按合并模板结构生成 plan.md，包含：

1. **Header**：Feature 名称、分支、Spec 链接
2. **Technical Context**：语言/版本、主要依赖、存储、项目结构、约束
3. **Constitution Check**：逐条核对
4. **文件结构**：已有文件（修改）+ 新增文件（创建）
5. **方案说明**：基于 spec.md Assumptions 中选择的方案，简要说明实现策略
6. **Tasks**：按阶段列任务清单，每个实现任务内嵌 TDD 步骤

### Task Right-Sizing

- **Task** = 最小自有测试周期 + 值得独立 reviewer gate 的单元
- **Step** = 一个动作（2-5 分钟）

### TDD 步骤模式（与 /tc-tdd 对齐）

每个实现任务包含显式 TDD 步骤：

```
- [ ] T0XX [P?] [US?] 简述 + 文件路径
  - Files: [涉及的所有文件]
  - Steps:
    1. RED: 在 tests/path/test_name.ext 写失败测试，验证 [具体行为]
    2. Verify RED: 运行测试，确认因功能缺失而失败（非拼写错误）
    3. GREEN: 在 src/path/file.ext 写最小实现代码通过测试
    4. Verify GREEN: 运行测试，确认通过且其他测试不受影响
    5. REFACTOR: 清理代码（消除重复、改善命名），保持测试通过
    6. Commit: git commit -m "T0XX [US#]: [简述]"
```

### Task 编号与标签规则

- **T0XX**：顺序编号，执行顺序即编号顺序
- **[P]**：仅用于可并行任务（不同文件、无未决依赖）
- **[US#]**：用户故事阶段任务必须带标签，Setup/Foundational/Polish 不带

### 阶段结构

- Phase 1: Setup（项目初始化、依赖安装、配置文件）
- Phase 2: Foundational（阻塞所有用户故事的前置基础设施）
- Phase 3+: 用户故事阶段（按 spec.md 优先级 P1/P2/P3 各成一阶段）
- Final Phase: Polish & Cross-Cutting（文档更新、性能优化、安全加固、跨故事重构）

### 禁止占位符

以下写法是 **plan failure**（不可出现在 plan.md 中）：
- TBD / TODO / 待定
- "add appropriate error handling"（必须写出具体处理什么错误、如何处理）
- "implement validation"（必须写出验证什么字段、什么规则）
- "write tests"（必须写出测试什么行为、在什么文件）
- 任何无法在 2-5 分钟内完成的步骤描述

## Phase 3: Research

仅当 Technical Context 有 NEEDS CLARIFICATION 项时进入。

1. 对每个未知项，研究并整合发现
2. 写 `research.md`（Decision / Rationale / Alternatives Considered 格式）
3. 更新 plan.md Technical Context 中的已解决答案

## Phase 4: Design & Contracts

按 speckit.plan Phase 1 模式：

1. **data-model.md**：从 spec 提取实体 → 实体字段、关系、校验规则、状态流转
2. **contracts/**：定义接口契约（如存在外部接口）
3. **quickstart.md**：写本地验证流程

## Phase 5: 自检

参考 `__ASSET_DIR__/references/plan-self-review.md`。

在 plan.md 写完后执行自检：

| 检查项 | 描述 |
|--------|------|
| Spec 覆盖 | 每个 FR-ID 在 Tasks 中至少有一个任务；每个 User Story 有对应阶段 |
| 占位符残留 | plan.md 无 TBD/TODO/[NEEDS CLARIFICATION:] 残留；每个步骤是具体可执行动作 |
| 类型一致性 | 实体名、字段名、接口 ID 在 data-model.md / plan.md / Tasks 中一致 |
| 宪法对齐 | Technical Context 与 Constitution Check 一致 |
| TDD 覆盖 | 每个实现任务有 RED/GREEN/REFACTOR 步骤 |
| 任务粒度 | 每个任务是最小自有测试周期单元；每步 2-5 分钟 |

发现问题则修复后重新自检，最多 3 轮。

## Phase 6: 执行交接

plan.md 写完并自检通过后，提议执行方式：

"(A) 自动逐任务执行(TDD 强制) — 类似 /tc-quick 的自动模式 / (B) 逐阶段确认执行 — 类似 /tc-flow / (C) 仅产出文件，手动继续"

- A → 逐阶段执行所有任务，强制 TDD，完成后在 plan.md 标记 `[X]`
- B → 每阶段完成后暂停确认，再进下一阶段
- C → 报告完成状态、文件路径、建议下一步

## 收尾：after_plan hook 检查

检查 `.specify/extensions.yml` 的 `hooks.after_plan`，规则同 before_plan 节。

## 完成报告

报告：分支、plan.md 路径、各 User Story 任务数、并行机会、自检结果。

## 红线

绝不：
- 写任何 TBD/TODO/占位符到 plan.md 中
- 让一个 Task 覆盖整个 feature——必须拆成最小自有测试周期单元
- 跳过 TDD 步骤模式——每个实现任务必须有 RED/GREEN/REFACTOR
- 在 plan.md 自检未通过前就开始执行
