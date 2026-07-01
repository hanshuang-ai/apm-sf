# plan.md 自检清单

## 使用方式

tc-writing-plans 在写出 plan.md 后自动执行本清单。

## 检查项

### A. Spec 覆盖
- [ ] 每个 FR-ID 在 Tasks 中至少有一个任务覆盖
- [ ] 每个 User Story (P1/P2/P3) 在 Tasks 中有对应的阶段
- [ ] spec.md 的所有 Key Entities 在 data-model.md 中出现（如有）

### B. 占位符残留
- [ ] plan.md 无 TBD / TODO / [NEEDS CLARIFICATION:] 残留
- [ ] Tasks 无 "add appropriate error handling" / "implement validation" / "write tests" 等空泛指令
- [ ] 每个任务步骤是具体、可执行的动作（2-5 分钟可完成）

### C. 类型一致性
- [ ] data-model.md（如有）中的实体名与 spec.md Key Entities 一致
- [ ] Tasks 中的文件路径与 plan.md 文件结构章节一致
- [ ] 字段命名风格在所有文档中统一

### D. 宪法对齐
- [ ] plan.md Technical Context 与 Constitution Check 一致
- [ ] Constitution Gate 未通过且有正当理由的，在 Complexity Tracking 中记录

### E. TDD 覆盖
- [ ] 每个实现任务有 RED/GREEN/REFACTOR 步骤
- [ ] 测试文件路径在 task 步骤中明确指定
- [ ] RED 步骤明确描述期望的失败原因

### F. 任务粒度
- [ ] 每个任务是最小自有测试周期的单元
- [ ] 每个步骤是一个动作（不是"实现功能"这种空泛描述）
- [ ] [P] 标记的任务确实不共享文件依赖
