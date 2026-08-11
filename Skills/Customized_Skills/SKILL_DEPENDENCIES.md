# Skill 依赖关系

## 全量依赖表

| Skill | 标识 | 直接依赖 | 被以下 skill 直接依赖 | 说明 |
| --- | --- | --- | --- | --- |
| `codebase-design` | `独立` | 无 | `improve-codebase-architecture` | 提供深模块、接口、接缝和适配器等共享架构词汇。 |
| `domain-modeling` | `独立` | 无 | `grill-with-docs`、`improve-codebase-architecture` | 维护领域词汇，并在满足条件时记录 ADR。 |
| `grilling` | `独立` | 无 | `grill-with-docs`、`improve-codebase-architecture` | 通过设计树和分轮提问澄清方案与决策。 |
| `grill-with-docs` | `组合` | `grilling`、`domain-modeling` | 无 | 使用 `grilling` 推进访谈，使用 `domain-modeling` 更新术语并生成 ADR。 |
| `handoff` | `独立` | 无 | 无 | 生成跨会话交接文档，只建议后续 skill，不执行它们。 |
| `improve-codebase-architecture` | `组合` | `codebase-design`、`grilling`、`domain-modeling` | 无 | 使用架构词汇扫描候选项，随后通过访谈和领域建模推进选中的方案。 |
| `to-spec` | `独立` | 无 | 无 | 将已确认的讨论与代码库信息整理为 Spec。读取领域词汇和 ADR 属于项目资料消费，不构成对 `domain-modeling` 的调用。 |
| `to-tickets` | `独立` | 无 | 无 | 将计划、Spec 或讨论拆分为带阻塞关系的纵向切片 Ticket。 |
