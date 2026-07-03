---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 前置检查

```bash
# 构建状态
qtcloud-devops build status

# 测试状态
qtcloud-devops test status

# 发布状态（最新 tag、版本一致性、CHANGELOG）
qtcloud-devops release status
```

确认版本一致、CHANGELOG 已更新、工作区干净后，进行发布。

## 预发布版本（rc）

发布 rc 版本用于 CI 验证：

```bash
qtcloud-devops release publish -v <scope>/v<version>-rc.<n> -y
```

例：

```bash
qtcloud-devops release publish -v rust/v0.1.5-rc.1 -y
```

CI 验证通过后，检察发布状态确认一致，再发布正式版本。

## 正式发布

```bash
qtcloud-devops release publish -v <scope>/v<version> -y
```

例：

```bash
qtcloud-devops release publish -v rust/v0.1.5 -y
```

发布流程：

1. 校验版本号格式
2. 校验所有配置文件版本号一致
3. 自动更新 `Cargo.toml`/`pyproject.toml` 版本号
4. 自动生成 CHANGELOG 条目（基于 git 提交记录，LLM 协助）
5. 校验 CHANGELOG 包含对应版本记录
6. 创建标签 → 推送到远端 → 创建 GitHub Release
7. CI 自动发布到 crates.io / PyPI

## 发布后检查

```bash
qtcloud-devops release status
```

确认最新标签已更新、CHANGELOG 与 GitHub Release 一致。

## 验收标准

- 配置文件与 Git tag 的版本号保持一致
- CHANGELOG 与 GitHub Release 存在并保持一致
- CI 构建全部通过

## 版本命名规范

以 Git tag 为事实源。完整规范见 `docs/handbook/lifecycle/release.md`：

- `scope/vX.Y.Z` — 正式版本（如 `rust/v0.1.5`）
- `scope/vX.Y.Z-prerelease` — 预发布版本（如 `cli/v0.3.2-rc.1`）
