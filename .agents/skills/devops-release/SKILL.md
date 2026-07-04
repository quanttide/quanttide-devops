---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 版本命名规范

以 Git tag 为事实源。版本号格式：

| 格式 | 说明 | 示例 |
|------|------|------|
| `scope/vX.Y.Z` | 带作用域的正式版本（monorepo 推荐） | `cli/v0.8.3`、`rust/v0.1.5` |
| `scope/vX.Y.Z-prerelease` | 预发布版本 | `cli/v0.8.3-rc.1`、`rust/v0.1.5-alpha.2` |
| `vX.Y.Z` | 无作用域版本（单包仓库用） | `v0.1.0` |

scope 对应 `.quanttide/devops/contract.yaml` 中定义的组件名。无 contract.yaml 时自动从仓库结构推断（`src/*`、`packages/*`、`apps/*`）。

## 自举流程

qtcloud-devops-cli 使用自身管理发布。流程如下：

### 1. 前置检查

```bash
# 构建状态
qtcloud-devops build status

# 测试状态
qtcloud-devops test status

# 发布状态（最新 tag、版本一致性、CHANGELOG）
qtcloud-devops release status
```

确认以下全部通过：

- 构建成功（`cargo check`）
- 测试通过（`cargo test`）
- 版本一致（配置文件版本与最新 tag 一致）
- CHANGELOG 已包含当前版本记录
- 工作区干净

### 2. 预发布版本（可选）

CI 验证用 rc 版本：

```bash
qtcloud-devops release publish -v cli/v0.8.3-rc.1 -y
```

等待 CI 运行完成，检查构建和测试通过后，再发正式版。

### 3. 正式发布

```bash
qtcloud-devops release publish -v cli/v0.8.3 -y
```

`release publish` 自动完成：

1. 校验版本号格式
2. `update_config_version` — 自动更新 Cargo.toml / pyproject.toml 版本号
3. 校验所有配置文件版本号一致
4. `ensure_changelog` — 自动生成 CHANGELOG 条目（git log → LLM 摘要 → 写入）
5. 校验 CHANGELOG 包含对应版本记录
6. `create_tag` + `push_tag` — 创建并推送 Git tag
7. `create_release` — 创建 GitHub Release

### 4. 发布后检查

```bash
qtcloud-devops release status
```

确认：

- 最新 tag 已更新
- CHANGELOG 与 GitHub Release 内容一致

## 注意事项

### CHANGELOG 只由 `release publish` 管理

不要手动修改 CHANGELOG.md。`ensure_changelog` 会自动检测版本是否存在，不存在时通过 LLM 生成。手动写入容易造成重复条目。

可视化步骤
已经修改的，通过发布后的检查

如果 CHANGELOG 已有该版本条目，`release publish` 会跳过生成阶段直接进入 tag/release。如需修正内容，在发布后编辑 GitHub Release body 即可。

### 发布可能超时

`release publish` 在 `push_tag` 或 `create_release` 阶段可能因网络阻塞卡住。

**如超时**：
1. 检查 tag 是否已推送：`git ls-remote --tags origin cli/v0.8.3`
2. 检查 GitHub Release 是否已创建：`gh release view cli/v0.8.3`
3. tag 已推送但 release 未创建 → 手动补：`gh release create cli/v0.8.3 --title "cli/v0.8.3" --notes "..."`

### 版本号 side effect

`update_config_version` 在预检前修改配置文件版本号。如果发布中途失败：

```bash
# 版本号已被改为目标版本，但实际未发布
git checkout -- Cargo.toml pyproject.toml  # 恢复原版本
```

修复问题后重新走完整发布流程。

### CHANGELOG 生成依赖 LLM

`ensure_changelog` 需要 `LLM_API_KEY` 环境变量。未配置时降级为报错，发布会继续但 CHANGELOG 条目可能缺失。

```bash
# 本地运行时确保配置
export LLM_API_KEY=sk-xxx
```

## CI 工作流

| Workflow | 触发 | 行为 |
|----------|------|------|
| `build-cli` | release published + tag `cli/*` | 版本校验 → 三平台构建 → wheel 构建 |
| `publish-cli` | build-cli 成功后 | publish-crate + publish-pypi |

## 常见问题

### CI 不触发

监听 `release: [published]` 事件。tag 已存在时不会重新触发。

**修复**：删除 release → `gh release create` 重新创建。

### 覆盖率未达标

```bash
cargo llvm-cov --lcov --output-path target/coverage/lcov.info
# 查看报告，补充测试
```

### 手动修复后重新发布

```bash
# 1. 删除远端 tag
git push --delete origin cli/v0.8.3

# 2. 删除 GitHub Release
gh release delete cli/v0.8.3 --yes

# 3. 修复后重新发布
qtcloud-devops release publish -v cli/v0.8.3 -y
```
