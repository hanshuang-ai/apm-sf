---
name: tc-quick
description: 小功能快速开发，可只给一句需求直接开干。已有 spec.md 时基于它生成精简的「计划+任务」合并文件 plan.md；spec.md 缺失且提供了需求时，自动调用 tc-specify 创建 spec 后继续；随后全自动逐任务执行（强制 TDD）。适用于无需分阶段评审的小功能；复杂功能仍用 /tc-flow 或原生四命令。
assets:
  - template.md
---

## 用户输入

```text
$ARGUMENTS
```

如非空，必须在执行前纳入考量。

## 前置：before_plan hook 检查

检查项目根的 `.specify/extensions.yml`：
- 不存在或无 `hooks.before_plan` 条目 → 跳过本节。
- 解析失败 → 静默跳过。
- 过滤 `enabled: false` 的 hook（无 `enabled` 字段视为启用）。
- 不解释 `condition` 表达式：无 condition 视为可执行；有非空 condition 则交由 HookExecutor，跳过。
- 对每个可执行 hook，按 `optional` 输出 Optional Pre-Hook（提示）或 Automatic Pre-Hook（`EXECUTE_COMMAND: {command}`，等待结果后再继续）。

## 流程

1. **路径解析**：从仓库根运行
   - sh：`scripts/bash/check-prerequisites.sh --paths-only --json`
   - ps：`scripts/powershell/check-prerequisites.ps1 -PathsOnly -Json`

   解析出 `FEATURE_DIR`、`FEATURE_SPEC`、`IMPL_PLAN`、`BRANCH`（均为绝对路径）。注意：该脚本要求当前在 feature 分支上，不在则会报错——这归入下面判定的「解析失败」分支。

2. **判定 spec 状态并按需自举**（决策矩阵）：

   | 状态 | 行为 |
   |---|---|
   | 第1步成功 **且** `FEATURE_SPEC`（spec.md）存在 | 直接进入第 3 步；`$ARGUMENTS` 作为补充上下文纳入考量 |
   | 第1步成功 **但** spec.md 缺失，且 `$ARGUMENTS` 非空 | 执行「自举」（见下），然后进入第 3 步 |
   | 第1步**失败**（不在 feature 分支 / 无 feature），且 `$ARGUMENTS` 非空 | 执行「自举」（见下），**重新运行第 1 步**拿到新路径后进入第 3 步 |
   | spec 缺失 / 无 feature **且** `$ARGUMENTS` 为空 | 停止，提示：请提供一句需求，或先运行 /tc-specify 生成 feature 结构 |

   **自举**：在本回合内**读取并遵循 tc-specify 命令的指令**，以 `$ARGUMENTS` 为功能描述创建 spec，附加两条覆盖：
   - **不停顿**：遇模糊点一律按行业默认/上下文合理推测，假设记入 spec 的 Assumptions 段，不向用户提澄清问题。
   - **精简**：跳过 tc-specify 的「Specification Quality Validation」环节（不创建 `checklists/requirements.md`、不做自校验迭代）。

   其余照 tc-specify 原样：生成短名、建分支（经 before_specify git 扩展 hook）、建 `specs/<dir>/`、拷 spec 模板填写、写 `.specify/feature.json`、执行 after_specify hook。

   自举后**重新运行第 1 步**；若仍报「Not on a feature branch」（通常因项目未装 git 扩展、分支未建），停止并输出清晰报错：提示安装 git 扩展或手动创建 feature 分支后重试，不要静默卡住。

3. **加载上下文**：读取 `FEATURE_SPEC`、`/memory/constitution.md`，以及合并骨架模板 `__ASSET_DIR__/template.md`。

4. **生成合并 plan.md**：以模板为结构，写入 `IMPL_PLAN`（即 `plan.md`）：
   - 填 Technical Context（未知项标 `NEEDS CLARIFICATION`）。
   - 填 Constitution Check。**Constitution Gate**：存在违反且无正当理由 → 报错并停止，不进入执行。
   - 从 spec.md 的用户故事（P1、P2…）生成 `## Tasks`，严格遵循 checklist 格式 `- [ ] T001 [P] [US1] 描述 + 文件路径`，按 Setup / Foundational / 用户故事（按优先级）/ Polish 分阶段。

5. **全自动执行**（不停顿）：
   - 逐阶段执行，完成每个阶段再进下一阶段。
   - **强制 TDD**：编写任何实现代码前，必须先调用并遵循 `tc-tdd` 技能，先写失败测试。
   - 按 plan.md 技术栈创建/校验 ignore 文件（.gitignore 等）。
   - 尊重依赖：顺序任务按序执行，[P] 任务可并行；同文件任务串行。
   - 每完成一个任务在 plan.md 的 `## Tasks` 中把该项标记为 `[X]`。
   - 每任务后上报进度；非并行任务失败即停并给出原因与下一步建议。

6. **完成校验**：所有任务完成且标 `[X]`；实现与 spec、plan、测试一致。

## 收尾：after_implement hook 检查

检查 `.specify/extensions.yml` 的 `hooks.after_implement`，规则同 before_plan 节：按 `optional` 输出 Automatic Hook（`EXECUTE_COMMAND: {command}`）或 Optional Hook（提示）。必须在向用户报告完成前处理本节（例如触发 update-progress）。

## 完成报告

报告分支、`plan.md` 路径、完成的任务数与最终状态摘要。

## Done When

- [ ] 已生成合并 plan.md（精简计划 + `## Tasks`），Constitution Gate 通过
- [ ] 所有任务完成并在 plan.md 标 `[X]`，实现与 spec/plan 一致、测试通过
- [ ] before_plan / after_implement hook 按规则触发或跳过
- [ ] 已向用户报告完成摘要
