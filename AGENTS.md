# AGENTS.md

## 子模组

本项目使用 Git 子模组管理关联仓库：

| 子模组 | 路径 | 用途 |
|--------|------|------|
| qtcloud-devops | `apps/qtcloud-devops` | DevOps 云平台应用 |
| devops-toolkit | `packages/toolkit` | DevOps 工具包 SDK |

**子模组操作规范：**
- 拉取更新：`git submodule update --remote <path>`
- 修改子模组：进入子模组目录提交后，回到主仓库更新引用
- 初始化克隆：`git clone --recurse-submodules <repo-url>`
