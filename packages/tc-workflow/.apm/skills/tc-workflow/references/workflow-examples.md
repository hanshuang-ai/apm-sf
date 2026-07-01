# workflow examples

本文件列出 `tc-workflow` 随技能默认装配的 6 个正式 preset。每个 preset 也有同名 `.yml` 文件，可直接运行，或复制到 `.specify/tc/workflows/` 后按项目调整。

## brainstorm-quick

深度设计并批准 `spec.md` 后，快速自动实现。

内置文件：`../assets/brainstorm-quick.yml`

核心流程：

```text
tc-brainstorm -> tc-quick
```

适合：需求需要认真澄清，但实现风险较低，希望 spec 批准后自动推进。

## web-design-build

使用 Visual Companion 做网页设计，再快速实现、评审、E2E。

内置文件：`../assets/web-design-build.yml`

核心流程：

```text
tc-brainstorm(visual_companion) -> tc-quick -> tc-review -> tc-e2e
```

适合：网页、后台页面、交互体验、布局方案需要先可视化比较的功能。

## native-speckit-review

原生 speckit specify → clarify → plan → tasks → implement 后追加评审。

内置文件：`../assets/native-speckit-review.yml`

核心流程：

```text
tc.specify -> tc.clarify -> tc.plan -> tc.tasks -> tc.implement -> tc-review
```

注意：`tc.clarify` 依赖已经生成的 `spec.md`，因此不能放在 `tc.specify` 之前。

## bugfix-artifact-repair

记录 bug，修补 `spec.md` / `plan.md` / `tasks.md`，并验证规格产物一致性。

内置文件：`../assets/bugfix-artifact-repair.yml`

核心流程：

```text
tc-bugfix-report -> tc-bugfix-patch -> tc-bugfix-verify -> note
```

适合：先修正规格产物和任务，再决定是否进入实现。

## bugfix-native-implement

进阶 bugfix：切换上下文、记录 bug、修补规格产物、验证、实现、评审。

内置文件：`../assets/bugfix-native-implement.yml`

核心流程：

```text
tc-bugfix-switch(optional) -> tc-bugfix-report -> tc-bugfix-patch -> tc-bugfix-verify -> tc.implement -> tc-review
```

注意：`tc-bugfix-switch` 会切分支并更新 feature context，因此该阶段是 `optional: true`、`mode: confirm`、`on_failure: ask`。

## brownfield-onboard

既有项目接入 tkit/speckit：扫描现状、生成项目定制配置、迁移既有功能规格产物，并校验配置。

内置文件：`../assets/brownfield-onboard.yml`

核心流程：

```text
tc-brownfield-scan -> tc-brownfield-bootstrap -> tc-brownfield-migrate -> tc-brownfield-validate
```

注意：brownfield 项目差异大，bootstrap 和 migrate 阶段都使用 `mode: confirm` 并带用户审批 gate；preset 不强行假设一定会产出 `spec.md`、`plan.md` 或 `tasks.md`。
