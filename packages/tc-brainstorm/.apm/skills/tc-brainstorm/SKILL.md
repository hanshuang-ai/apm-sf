---
name: tc-brainstorm
description: 需要深度脑暴设计一个功能时使用。探索项目上下文、逐步澄清需求、提出 2-3 种方案并附带权衡对比、逐节审批后写出 spec-kit 标准的 spec.md。批准后交给 /tc-quick 快速计划并实现。
assets:
  - references/brainstorm-checklist.md
  - references/visual-companion.md
  - scripts/frame-template.html
  - scripts/helper.js
  - scripts/server.cjs
  - scripts/start-server.sh
  - scripts/stop-server.sh
---

# Brainstorming Ideas Into Designs

把模糊的想法通过协作式对话变成完整的 spec-kit 标准 `spec.md`。

## 用户输入

```text
$ARGUMENTS
```

如非空，必须在执行前纳入考量。空输入时从项目上下文推断；如果仍无法确定功能目标，停止并要求用户提供功能描述。

## 硬门禁

<HARD-GATE>
在用户批准设计和 `spec.md` 之前，不得编写任何代码、搭建任何项目或采取任何实现行动。这适用于每个项目，无论感知的简单程度如何。
</HARD-GATE>

## 反模式："这太简单不需要设计"

每个项目都走完整流程。一个 todo list、一个单函数工具、一个配置修改——全都一样。"简单"项目是最容易因未审视的假设导致返工的地方。设计可以很短（真正简单的项目几句话就行），但必须呈现并获得批准。

## Checklist

必须按顺序完成：

1. **Explore project context** — 检查文件、文档、最近提交、spec-kit 上下文
2. **Offer Visual Companion** — 如果后续问题涉及视觉内容，单独询问是否启用浏览器辅助
3. **Ask clarifying questions** — 一次只问一个问题，理解目的、约束和成功标准
4. **Propose 2-3 approaches** — 用对话方式说明权衡和推荐方案
5. **Present design** — 按复杂度分节呈现设计，并逐节获得用户确认
6. **Write spec.md** — 写入 spec-kit 的 `FEATURE_SPEC`
7. **Spec self-review** — 自检占位符、矛盾、歧义、范围和 spec-kit 一致性
8. **User reviews written spec** — 要求用户审阅 `spec.md`
9. **Transition to implementation** — 执行 after_specify hooks 后交给 `/tc-quick`

## 前置：before_specify hook 检查

检查项目根的 `.specify/extensions.yml`：

- 不存在或无 `hooks.before_specify` 条目：跳过本节。
- 解析失败：静默跳过。
- 过滤 `enabled: false` 的 hook；无 `enabled` 字段视为启用。
- 对每个可执行 hook，按 `optional` 输出 Optional Pre-Hook 提示，或输出 Automatic Pre-Hook 的 `EXECUTE_COMMAND: {command}` 并等待结果后再继续。

## Phase 0: 上下文探索 + 特性目录准备

1. **路径解析**：从仓库根运行可用的 spec-kit prerequisite 脚本。
   - sh：`scripts/bash/check-prerequisites.sh --paths-only --json`
   - ps：`scripts/powershell/check-prerequisites.ps1 -PathsOnly -Json`
   - 解析 `FEATURE_DIR`、`FEATURE_SPEC`、`BRANCH` 等路径。

2. **判定特性目录状态**：

   | 状态 | 行为 |
   |------|------|
   | `FEATURE_DIR` 存在且 `spec.md` 存在 | 复用目录，加载已有 spec，进入修订模式 |
   | `FEATURE_DIR` 存在但 `spec.md` 缺失 | 复用目录，跳过分支创建，写新 spec.md |
   | 无特性目录且 `$ARGUMENTS` 非空 | 调用 create-new-feature 脚本创建分支和目录 |
   | 无特性目录且 `$ARGUMENTS` 为空 | 停止，提示用户提供功能描述 |

3. **扫描项目上下文**：
   - 读取可用的项目文档，如 `docs/project/requirements.md`、`docs/project/architecture.md`、`docs/project/feature-breakdown.md`。
   - 读取最近提交：`git log --oneline -10`。
   - 读取 `.specify/memory/constitution.md` 获取约束。

4. **检测基线/brief 状态**：
   - 检查 `docs/project/formalize-status.md`。
   - `handoff-ready` 时，可把 `docs/project/briefs/<feature-id>-specify-input.md` 作为上下文种子。
   - `baseline-only` 时，把项目级文档作为上下文。
   - `blocked` 时，提醒用户先解决阻断项。
   - 无状态文件时，独立使用用户输入和代码库扫描结果。

基线或 brief 只能减少重复说明，不能替代后续自然澄清流程。

## Phase 1: Offer Visual Companion

如果接下来的问题会涉及 mockup、布局、架构图、关系图、侧-by-side 视觉比较或设计打磨，必须先单独发送以下消息，不附带任何其他内容：

> Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)

如果用户同意，读取 `__ASSET_DIR__/references/visual-companion.md`，并按其中的浏览器辅助流程执行。Visual Companion 是工具，不是模式；每个问题都要独立判断用浏览器还是终端。

如果问题本质是文本、范围、概念、技术权衡或需求选择，继续在终端对话，不要为了视觉话题本身强行启用浏览器。

## Phase 2: Ask Clarifying Questions

一次只问一个问题。优先多选，但当用户需要解释目的或约束时使用开放问题。

重点理解：

- 功能目的和成功标准
- 范围和明确不做的内容
- 用户角色和核心场景
- 关键数据或实体
- 业务规则、约束和状态变化
- 边界情况和错误处理
- 性能、安全、兼容性等非功能要求

如果需求覆盖多个独立子系统，例如用户管理、支付、通知同时出现，先停止详细澄清并帮助拆分：说明哪些子项目相互独立、依赖关系是什么、建议先脑暴哪一个。每个子项目都应单独形成 spec、plan、implementation cycle。

## Phase 3: Propose 2-3 Approaches

基于已澄清的需求，提出 2-3 种可行方案。用自然对话说明：

- 每种方案的核心思路
- 对现有结构的影响
- 优势和劣势
- 复杂度和主要风险
- 对用户体验或维护成本的影响

先给出推荐方案，并说明为什么它最适合当前代码库和目标。询问用户这个方向是否正确；如用户调整方向，修订方案后再继续。

## Phase 4: Present Design

在写 `spec.md` 之前，按分节呈现设计并获得用户确认。简单功能可以每节几句话；复杂功能每节最多约 200-300 字。

建议覆盖：

- Scope and non-scope
- User scenarios
- Architecture and component boundaries
- Data flow and key entities
- Error handling and edge cases
- Testing and success criteria

每节之后询问用户是否看起来正确。用户不同意时，返回澄清或修订对应设计，不继续写 spec。

### 设计隔离与清晰性

- 把系统拆成更小的单元，每个单元有单一清晰用途。
- 单元之间通过定义良好的接口通信，能独立理解和测试。
- 对每个单元应能回答：它做什么、怎么用、依赖什么。
- 如果不看内部无法理解一个单元，或改内部会破坏消费者，边界需要调整。

### 现有代码库中的工作

- 提出变更前先探索当前结构，遵循已有模式。
- 如果现有代码存在影响当前工作的边界或职责问题，在设计中包含有针对性的改善。
- 不提出无关重构。

## Phase 5: Write spec.md

用户批准设计后，写入 `FEATURE_SPEC`，通常是 `specs/<feature>/spec.md`。按 spec-kit 的 `spec-template.md` 结构组织：

1. **Header**：Feature 名称、分支、日期、状态
2. **User Scenarios & Testing**：P1/P2/P3 User Stories、Given/When/Then 验收、Edge Cases
3. **Requirements**：FR-NNN MUST 语句、Key Entities
4. **Success Criteria**：SC-NNN，可测量且技术无关
5. **Assumptions**：假设、批准的设计方向、范围排除项

`tc-brainstorm` 的产物是 spec-kit `spec.md`，不是 `docs/superpowers/specs/...-design.md`。

## Phase 6: Spec Self-Review

参考 `__ASSET_DIR__/references/brainstorm-checklist.md`。写出完整 `spec.md` 后自检：

- 无 TBD、TODO 或未解决澄清标记。
- User Stories、Requirements、Success Criteria 和 Assumptions 不互相矛盾。
- 范围足够聚焦，能作为一个 feature 实现。
- 每条 FR 可以唯一解读。
- FR 和 SC 编号内部一致。
- 不违反 `.specify/memory/constitution.md`。
- spec 反映用户批准的设计方向。

发现问题则直接修复后重新自检。

## Phase 7: User Reviews Written Spec

自检通过后，要求用户审阅文件：

> Spec written to `<path>`. Please review it and let me know if you want to make any changes before we hand it to `/tc-quick`.

如果用户要求修改，更新 `spec.md` 并重新自检。只有用户批准后才能继续。

## Phase 8: /tc-quick Handoff

用户批准 `spec.md` 后：

1. 执行 after_specify hooks，与 `/speckit.specify` 保持一致。
2. 报告分支名、`spec.md` 路径、自检结果和批准的设计方向。
3. 调用 `/tc-quick`，基于当前 `spec.md` 快速生成精简 `plan.md` 并自动 TDD 实现。

**终端状态是交给 `/tc-quick`。** 不在 `tc-brainstorm` 内提供其他实现路线菜单。

## 完成报告

报告：

- 分支名
- `spec.md` 路径
- 自检结果
- 批准的设计方向
- 已交给 `/tc-quick`

## 红线

绝不：

- 在设计和 `spec.md` 获批前写任何代码或调用实现类技能。
- 跳过逐题澄清、方案对比、逐节设计确认、spec 自检或用户审阅。
- 把基线/brief 输入当作用户最终批准。
- 把 Visual Companion 降级为 Mermaid-only 协议；网页、UI、交互、布局和图形化比较仍使用浏览器辅助。
- 在用户回复"暂停"或要求修改时继续推进到 `/tc-quick`。
