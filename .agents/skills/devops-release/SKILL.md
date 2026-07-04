---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 版本号规则

以 Git tag 为事实源。格式：

- `scope/vX.Y.Z` — monorepo 正式版，如 `cli/v0.8.3`
- `scope/vX.Y.Z-prerelease` — 预发布版，如 `cli/v0.8.3-rc.1`
- `vX.Y.Z` — 单包仓库版

scope 对应 `.quanttide/devops/contract.yaml` 中的组件名。无配置时自动推断。

## 流程

### 1. 前置检查

```bash
qtcloud-devops status
```

统一 status 按生命周期顺序聚合输出：contract → doctor → plan → code → build → test → release。确认以下全部通过：

- 构建成功
- 测试全部通过
- 版本一致（配置文件版本与最新 tag 一致）
- 工作区干净

### 2. 发布

```bash
qtcloud-devops release publish -v <version> -y
```

`release publish` 内部自动完成：更新配置文件版本 → 生成 CHANGELOG → 创建 tag → 推送 → 创建 GitHub Release。

**所有步骤都是幂等的。** 超时或失败后直接重跑相同的 `publish` 命令，不会产生重复 tag 或 Release。

### 3. 发布后检查

```bash
qtcloud-devops status
```

或只看发布状态：

```bash
qtcloud-devops release status
```

## 常见问题

### CI 不触发

监听 `release: [published]` 事件。如果 CI 未触发，用 `--force` 重新发布：

```bash
qtcloud-devops release publish -v cli/v0.8.3 -y -f
```

`--force` / `-f` 会自动删除已存在的远端 tag 和 GitHub Release，然后重新创建。

### 版本号冲突需要重新发布

修复代码后，用 `--force` 重新发布：

```bash
qtcloud-devops release publish -v cli/v0.8.3 -y -f
```

等效于手动执行以下步骤，但一次完成：

- 删除远端 tag（`git push --delete origin cli/v0.8.3`）
- 删除 GitHub Release（`gh release delete cli/v0.8.3 --yes`）
- 重新创建 tag、推送、创建 Release
