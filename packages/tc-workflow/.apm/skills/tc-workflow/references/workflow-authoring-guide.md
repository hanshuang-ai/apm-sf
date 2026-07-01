# workflow authoring guide

当用户要求创建、复制、修改或修复 workflow 时，使用本指南。不要从零猜测完整 YAML；先判断用户意图，再选择最小必要问题。

## 意图分类

| 意图 | 行为 |
|------|------|
| 从零创建 | 根据用户目标设计新的 `.specify/tc/workflows/<name>.yml` |
| 复制内置 preset | 找到最接近的内置 preset，复制到项目 workflow 后修改 |
| 修改已有项目 workflow | 读取 `.specify/tc/workflows/<name>.yml` 或 `.yaml` 后做最小改动 |
| 修复失败 workflow | 结合 validate/run 错误、schema 和 troubleshooting 资料修复 |

## preset 选择

| 用户目标 | 优先基础 |
|----------|----------|
| 需求澄清后快速实现 | `../assets/brainstorm-quick.yml` |
| 网页、页面、交互体验设计并实现 | `../assets/web-design-build.yml` |
| 使用原生 speckit specify/clarify/plan/tasks/implement | `../assets/native-speckit-review.yml` |
| 只修补 bug 相关规格产物 | `../assets/bugfix-artifact-repair.yml` |
| bugfix 从记录到实现和评审 | `../assets/bugfix-native-implement.yml` |
| 既有项目接入 tkit/speckit | `../assets/brownfield-onboard.yml` |

## 最少提问

只问缺失且会影响 YAML 的问题：

1. workflow 名称是什么？
2. 起点是什么：从自然语言需求开始，还是已有 `spec.md` / `plan.md` / `tasks.md`？
3. 终点是什么：只生成规格产物，还是继续实现、评审、测试？
4. 哪些阶段需要人工确认？
5. 失败策略是什么：停止、询问，还是可选阶段允许继续？

如果用户输入已经回答这些问题，直接生成或修改 workflow。

## 写入规则

- 项目自定义 workflow 写入 `.specify/tc/workflows/<name>.yml`。
- 顶层 `name` 必须与文件名 `<name>` 一致。
- 每个 stage 必须有唯一 `id` 和合法 `kind`。
- `skill` / `command` stage 必须写 `use`。
- `shell` stage 必须写 `run`，且命令必须来自用户明确要求或同意。
- 依赖关系用 `requires` / `produces` 表达，不用隐式步骤补齐。
- 不发明 schema 外字段。

## 生成后检查

写完后建议用户运行：

```text
/tc-workflow validate <name>
```

如果校验失败，读取 `workflow-schema.md` 和 `workflow-troubleshooting.md` 后修复。
