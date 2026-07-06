---
name: devops-build
description: 量潮 DevOps 构建管理。当用户说"构建"、"build"、"编译"时使用。
---

# devops-build — 构建管理

## 命令

| 命令 | 用途 |
|------|------|
| `qtcloud-devops build status` | 查看构建状态（编译、依赖、版本一致性） |
| `qtcloud-devops build clean` | 清理构建产物（target/、dist/ 等） |
| `qtcloud-devops build audit` | 构建审计：检查编译器配置、CI 工作流、依赖声明 |

`build status` 和 `build audit` 是读操作，`build clean` 是写入操作。
