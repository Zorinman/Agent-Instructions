# Workflow
## 项目的开发采用 adr，specs,tickers,archives进行管理
- adr: Architecture Decision Records (架构决策记录) 用于记录项目中做出的重要架构决策，帮助团队理解为什么做出这些决策以及它们的影响。
- specs: 规范文档，用于详细描述项目的功能、接口、数据结构和其他技术细节，确保团队成员对项目的要求有一致的理解。
- tickers: 任务跟踪器，用于记录和跟踪项目中的任务、问题和进展，帮助团队管理工作流程和优先级。
- archives: 存档文件，用于保存项目的历史记录、旧版本和其他相关

## 某个功能的开发其 adr、specs、tickers、archives 文件的路径如下,且文件需要一一对应：
- adr: `.../workflow/docs/adr/`
- specs: `.../workflow/docs/specs/`
- tickers: `.../workflow/docs/tickers/`
- archives: `.../workflow/docs/archives/`
## 命名规范如下
序列编号_文档类型(adr/specs/tickers/archives)_功能名称.md
例如：
- 0001_adr_用户认证.md
- 编号从现有的最大编号开始递增（adr,specs,tickers,archives所有文档中最大的），确保唯一性。
## 归档
当某个功能的开发完成后且用户提出归档需求时，其对应的 adr、specs、tickers 文件将被移动到 archives 文件夹中进行归档，以便于后续查阅和管理。其中内容应加上归档记录,具体格式如下：
```markdown
---
title: 
status: 
depends_on: 
implemented_in:(如果用户已经git提交则写提交号,否则写worktree)
archived_on: 
---

- **状态**：已归档。
- **实现范围**：
- **实现位置**：
- **验证范围**：
```

同时将本次开发的功能更新到 `.../docs/New_Features.md` 中。

