# spec.md 自检清单

## 使用方式

tc-brainstorm 在写出 spec.md 后自动执行本清单。用户也可在审阅时参考。

## 检查项

### A. 占位符残留
- [ ] spec.md 中无 TBD / TODO / [NEEDS CLARIFICATION:] 残留（Assumptions 章节的合法登记除外）
- [ ] 所有 FR-NNN 和 SC-NNN 编号已填充具体内容，不是占位符

### B. 内部一致性
- [ ] Success Criteria 中引用的 FR-ID 在 Requirements 章节中存在
- [ ] User Stories 的验收场景覆盖了对应的 FR-ID
- [ ] Key Entities 与 User Stories 中提及的数据对象一致
- [ ] Assumptions 中记录的方案选择与 spec 正文的描述一致

### C. 范围检查
- [ ] spec 明确列出"本 feature 做什么"
- [ ] spec 明确列出"本 feature 不做什么"
- [ ] 无未讨论的范围蔓延（超出澄清阶段确认的范围）

### D. 歧义检查
- [ ] 每条 FR 可以被唯一解读（不存在两种合理实现路径）
- [ ] 验收场景的 Given/When/Then 无歧义

### E. 宪法对齐
- [ ] spec 不违反 `.specify/memory/constitution.md` 的原则
- [ ] 如有偏离，Assumptions 中有正当理由记录

### F. 设计方向对齐
- [ ] spec 内容反映用户批准的方案方向
- [ ] 未采用的重要方案或范围排除项已在 Assumptions 中说明
