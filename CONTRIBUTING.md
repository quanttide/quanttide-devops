# Contributing

## 子模组

本项目使用 Git 子模组管理关联仓库：

| 子模组 | 路径 | 用途 |
|--------|------|------|
| qtcloud-devops | `apps/qtcloud-devops` | DevOps CLI 工具 |
| qtcloud | `apps/qtcloud` | 量潮云平台 |
| devops-toolkit | `packages/quanttide-devops-toolkit` | DevOps 工具包 SDK |
| laboratory | `examples/default` | 实验室 |
| gallery | `docs/gallery` | 量潮 DevOps 案例 |
| handbook | `docs/handbook` | DevOps 实践手册 |
| tutorial | `docs/tutorial` | DevOps 教程 |
| essay | `docs/essay` | 技术随笔 |
| brochure | `data/brochure` | 宣传册 |
| bylaw | `data/bylaw` | 章程 |
| context | `data/context` | DevOps 实践上下文 |
| journal | `data/journal` | 开发日志 |
| report | `data/report` | 报告 |
| roadmap | `data/roadmap` | 路线图 |
| history | `data/history` | 工作历史 |

**子模组操作规范：**
- 拉取更新：`git submodule update --remote <path>`
- 修改子模组：进入子模组目录提交后，回到主仓库更新引用
- 初始化克隆：`git clone --recurse-submodules <repo-url>`
