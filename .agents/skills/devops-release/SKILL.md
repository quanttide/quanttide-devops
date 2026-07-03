---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 版本命名规范

以 Git tag 为事实源。版本号格式：

| 格式 | 说明 | 示例 |
|------|------|------|
| `scope/vX.Y.Z` | 带作用域的正式版本 | `rust/v0.1.5`、`python/v0.2.0` |
| `scope/vX.Y.Z-prerelease` | 带作用域的预发布版本 | `cli/v0.3.2-rc.1`、`rust/v0.1.5-alpha.2` |
| `vX.Y.Z` | 无作用域版本（较少用） | `v0.1.0` |

## 前置检查

发布前执行以下命令确认一切就绪：

```bash
# 1. 构建状态
qtcloud-devops build status

# 2. 测试状态
qtcloud-devops test status

# 3. 发布状态（显示最新 tag、版本一致性、CHANGELOG 状态）
qtcloud-devops release status
```

确认以下内容全部通过：

- 构建成功
- 测试通过
- 版本一致（配置文件版本与最新 tag 一致）
- CHANGELOG 已包含当前版本记录
- 工作区干净（无未提交的修改）

## 预发布版本（rc 版本）

发布 rc 版本用于 CI 验证：

```bash
# rc.1 首次预发布
qtcloud-devops release publish -v rust/v0.1.5-rc.1 -y

# rc.2 第二次预发布（如有修复）
qtcloud-devops release publish -v rust/v0.1.5-rc.2 -y
```

发布后等待 CI 运行完成：

- `test-rust` workflow 自动触发（监听 `release: [published]`）
- 检查构建、测试、覆盖率阈值（≥ 95%）全部通过

CI 验证通过后，检察发布状态确认一致，再发布正式版本。

## 正式发布

### 1. 执行发布

```bash
qtcloud-devops release publish -v rust/v0.1.5 -y
```

`release publish` 命令自动完成以下步骤：

1. 校验版本号格式（必须是 `scope/vX.Y.Z` 或 `vX.Y.Z`）
2. 校验所有配置文件版本号一致（Cargo.toml / pyproject.toml / package.json 等）
3. 自动更新配置文件版本号
4. 自动生成 CHANGELOG 条目（基于 git 提交记录，LLM 协助）
5. 校验 CHANGELOG 包含对应版本记录
6. 创建 Git tag 并推送到远端
7. 创建 GitHub Release

### 2. 等待 CI

tag 推送后自动触发两个 workflow：

| Workflow | 触发条件 | 内容 |
|----------|---------|------|
| `test-rust` | `release: [published]` + `startsWith(github.ref, 'refs/tags/rust/')` | build → test → coverage ≥ 95% |
| `publish-rust` | `release: [published]` + `startsWith(github.ref, 'refs/tags/rust/')` | `cargo publish` 发布到 crates.io |

在 GitHub 仓库的 Actions 页面查看运行状态：

```bash
gh run list --repo https://github.com/quanttide/quanttide-devops-toolkit \
  --workflow publish-rust --limit 3
```

### 3. 发布后检查

```bash
qtcloud-devops release status
```

确认：

- 最新 tag 已更新到刚刚发布的版本
- CHANGELOG 与 GitHub Release 内容一致
- crates.io / PyPI 上已存在对应版本

## 验收标准

- 配置文件中的版本号与 Git tag 保持一致
- CHANGELOG 条目存在且与 GitHub Release 内容一致
- `test-rust` CI：build + test + coverage ≥ 95% 全部通过
- `publish-rust` CI：`cargo publish` 成功
- crates.io / PyPI 上已发布对应版本

## 常见问题

### CI 不触发

`publish-rust` / `test-rust` workflow 监听 `release: [published]` 事件。如果 tag 已存在但指向错误 commit，仅 `git push --force` 不会重新触发 workflow。

**修复方法**：在 GitHub 上删除 release，然后重新用 `gh release create` 创建 release。

### `crate already exists` 错误

```
error: crate quanttide-devops@0.1.4 already exists on crates.io index
```

tag 指向的 commit 中 `Cargo.toml` 版本号未更新。确认 `Cargo.toml` 版本号正确后，删除 release 和远端 tag，更新版本号后重新打 tag 并创建 release。

### `could not find Cargo.toml`

CI 在错误目录执行。检查对应 workflow 文件中的 `defaults.run.working-directory` 是否正确（如 `packages/rust`）。

### 覆盖率低于 95%

```bash
# 本地复现
cargo llvm-cov --all-features --tests --lcov --output-path lcov.info
# 查看覆盖报告
# 补充测试直到达标
```

### 手动修复后重新发布

整个流程不可修改中间状态后重试，需要从头再来：

```bash
# 1. 删除远端 tag
git push --delete origin rust/v0.1.5

# 2. 删除 GitHub Release
gh release delete rust/v0.1.5 --repo https://github.com/quanttide/quanttide-devops-toolkit --yes

# 3. 修复问题后，重新执行发布
qtcloud-devops release publish -v rust/v0.1.5 -y
```
