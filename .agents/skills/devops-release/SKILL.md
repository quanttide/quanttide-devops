---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

你是发布流程的决策者，不是脚本执行器。
流程中每个环节都需要你收集状态、做出判断，而不是机械地敲命令。

## 你的工具

| 工具 | 用途 |
|------|------|
| `qtcloud-devops build status` | 查看构建状态 |
| `qtcloud-devops test status` | 查看测试状态 |
| `qtcloud-devops release status` | 查看发布状态（tag、版本一致性、CHANGELOG） |
| `qtcloud-devops release publish -v <version> -y` | 执行发布（自动完成校验→tag→push→GitHub Release） |
| `git` / `gh` | 异常恢复时手动操作 |

## 版本号规则

以 Git tag 为事实源。格式：

- `scope/vX.Y.Z` — monorepo 正式版，如 `cli/v0.8.3`
- `scope/vX.Y.Z-prerelease` — 预发布版，如 `cli/v0.8.3-rc.1`
- `vX.Y.Z` — 单包仓库版

scope 对应 `.quanttide/devops/contract.yaml` 中的组件名。无配置时自动推断。

## 流程

### 1. 收集状态（不要只跑命令，要评估）

执行以下三个工具获取当前状态：

- `qtcloud-devops build status`
- `qtcloud-devops test status`
- `qtcloud-devops release status`

**你的判断：**

| 条件 | 结论 |
|------|------|
| 构建失败 | **阻断**，不继续发布。告知用户失败详情 |
| 测试覆盖率低于阈值或测试失败 | **阻断**，不继续发布 |
| 版本不一致（配置文件 tag 不匹配） | **阻断**，需先同步版本 |
| 工作区有未提交变更 | **阻断**，需先提交或 stash |
| CHANGELOG 缺少当前版本记录 | **警告**，发布会自动生成，但你应告知用户 |
| 以上全过 | 进入下一步 |

### 2. 判断发布类型

收集用户意图，或根据版本号自动判断：

```
v1.0.0-rc.1, v1.0.0-alpha.1  → 预发布
v1.0.0, cli/v0.8.3            → 正式发布
```

**预发布**：向用户确认 → 执行发布 → 告知 CI 正在运行，等 CI 通过再发正式版。

**正式发布**：向用户呈现汇总信息——版本号、主要变更摘要、发布后影响——**让用户确认后再执行**。不要自动发正式版。

### 3. 执行发布

```bash
qtcloud-devops release publish -v <version> -y
```

`release publish` 内部自动完成：版本校验 → 更新配置文件版本 → 生成 CHANGELOG → 创建 tag → 推送 → 创建 GitHub Release。

**如果执行中发现 CHANGELOG 条目有问题**（分类不合理、遗漏重要变更）：
- 记录问题，告知用户
- 发布仍可完成（Release note 发完后可以编辑）
- 建议用户在 GitHub Release 页面上手动修正

**如果执行超时（网络问题）**：
- 检查 tag 是否已推送：`git ls-remote --tags origin <version>`
- 检查 Release 是否已创建：`gh release view <version>`
- tag 已推送但 Release 未创建 → `gh release create <version> --title "<version>" --notes "..."`

### 4. 发布后验证

```bash
qtcloud-devops release status
```

**确认：**
- 最新 tag 已更新为目标版本
- CHANGELOG 与 GitHub Release 内容一致
- CI 工作流已触发（在 GitHub Actions 中查看）

如果 CI 未触发（如 tag 已存在），告知用户：删除 Release 后用 `gh release create` 重新创建可重发事件。

## 边界情况处理指南

### 版本号已发布
如果 tag 和 Release 都已存在 → 告知用户版本已发布，无需重复操作。
如果 tag 存在但 Release 不存在 → 执行 `gh release create` 补创建。

### 发布中途失败
`release publish` 可能修改了配置文件的版本号但未完成 tag/release。
告知用户：`git checkout -- Cargo.toml pyproject.toml` 恢复版本号后重试。

### 用户要求手动修复后重新发布
流程：删除远端 tag → 删除 GitHub Release → 修复代码 → 重新发布。

```bash
git push --delete origin <version>
gh release delete <version> --yes
# 修复后
qtcloud-devops release publish -v <version> -y
```
