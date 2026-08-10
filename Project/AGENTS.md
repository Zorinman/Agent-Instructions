# 项目文档工作流

项目开发使用 ADR、Spec、Ticker 和 Archive 管理。一个功能按以下顺序推进：

1. `/Grill With Docs` 讨论设计并生成 ADR。
2. `/To Spec` 生成 Spec。
3. `/To Tickets` 将工作拆分为 Ticker 文件。
4. 根据用户提示开始实现具体代码。每完成Ticker中的一个task则同步更新Ticker文档。

该流程不是每次编码的必经步骤。是否采用全部或部分步骤由用户决定。

ADR 记录重要架构决策及其影响。Spec 描述功能、接口、数据结构和技术细节。Ticker 跟踪可执行任务、依赖关系和进度。Archive 保存已完成功能的文档快照。

本地 `workflow/docs/` 是正式文档来源。外部 issue tracker 可以同步 Spec 或 Ticket，但不替代本地文档。

## 功能文档组

一个功能由以下文档组成：

- 一份 ADR。
- 一份 Spec。
- 一组 Ticker 文件。

ADR、Spec 和 Ticker 组使用相同的功能编号与功能名称。Ticker 组中的每个任务文件使用独立的任务序号。

## 路径与命名

以功能编号 `0001`、功能名称 `用户认证` 为例：

```text
workflow/docs/
├── adr/
│   └── 0001_adr_用户认证.md
├── specs/
│   └── 0001_specs_用户认证.md
├── tickers/
│   └── 0001_tickers_用户认证/
│       ├── 00_概览.md
│       ├── 01_用户注册.md
│       ├── 02_用户登录.md
│       └── 03_认证验证.md
└── archives/
```

创建新功能时，扫描 `adr`、`specs`、`tickers` 和 `archives` 中已有的功能编号，取最大值加一。已有功能文档组时，后续 ADR、Spec 和 Ticker 必须复用该功能编号与功能名称。

Ticker 文件按依赖顺序从 `01` 开始编号。`00_概览.md` 记录功能编号、关联 ADR 与 Spec、Ticket 列表、依赖关系和整体状态。

## 归档

仅当功能已实现完成且用户明确要求归档时，归档对应的 ADR、Spec 和整个 Ticker 目录。归档后的目录结构如下：

```text
workflow/docs/archives/
└── 0001_用户认证/
    ├── ARCHIVE.md
    ├── 0001_adr_用户认证.md
    ├── 0001_specs_用户认证.md
    └── 0001_tickers_用户认证/
        ├── 00_概览.md
        ├── 01_用户注册.md
        ├── 02_用户登录.md
        └── 03_认证验证.md
```

`ARCHIVE.md` 使用以下格式：

```markdown
---
title: 用户认证
status: archived
feature_id: 0001
depends_on: []
implemented_in: worktree
archived_on: 2026-08-10
---

# 用户认证

- **状态**：已归档。
- **实现范围**：用户注册、登录与认证验证。
- **实现位置**：相关实现模块和测试。
- **验证范围**：自动化测试与已完成的验证流程。

## 文档

- [ADR](./0001_adr_用户认证.md)
- [Spec](./0001_specs_用户认证.md)
- [Tickers](./0001_tickers_用户认证/00_概览.md)
```

已提交代码时，`implemented_in` 填写提交号；未提交时填写 `worktree`。归档完成后，在 `docs/New_Features.md` 中增加功能名称、功能编号、归档日期和 `ARCHIVE.md` 链接。
