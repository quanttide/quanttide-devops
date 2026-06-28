# AGENTS.md

## 项目定位

`quanttide-devops` 是量潮科技的 DevOps 基础设施仓库，聚合所有 DevOps 相关的仓库为子模块，并提供统一的 CLI 工具。

## 文档体系

```
docs/   → 给人读的（文章型文档）
  essay/         随笔
  handbook/      手册（生命周期视角：规划→编码→构建→测试→发布→部署→运维→监控）
  tutorial/      教程（主题视角：快速入门→Scrum→Git→流水线→制品→OpenAPI→设计开发→CI/CD→发布）
data/   → 当数据用的（结构化记录）
  brochure/      宣传册
  journal/       日志
  report/        报告
  roadmap/       路线图
```

**核心划分原则：**
- handbook 按**流程**导航，回答"这个环节用什么"
- tutorial 按**主题**教学，回答"某个知识点怎么用"
- data/ 下的内容是结构化记录，不是文章

## 子模组

本项目使用 Git 子模组管理关联仓库：

| 子模组 | 路径 | 用途 |
|--------|------|------|
| qtcloud-devops | `apps/qtcloud-devops` | DevOps CLI 工具 |
| qtadmin | `apps/qtadmin` | 量潮管理后台 |
| devops-toolkit | `packages/toolkit` | DevOps 工具包 SDK |
| laboratory | `examples/default` | 实验室 |
| handbook | `docs/handbook` | DevOps 实践手册 |
| tutorial | `docs/tutorial` | DevOps 教程 |
| essay | `docs/essay` | 技术随笔 |
| brochure | `data/brochure` | 宣传册 |
| journal | `data/journal` | 开发日志 |
| report | `data/report` | 报告 |
| roadmap | `data/roadmap` | 路线图 |
| history | `data/history` | 工作历史 |

**子模组操作规范：**
- 拉取更新：`git submodule update --remote <path>`
- 修改子模组：进入子模组目录提交后，回到主仓库更新引用
- 初始化克隆：`git clone --recurse-submodules <repo-url>"`
