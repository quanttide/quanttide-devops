---
name: devops-release
description: 量潮 DevOps 包发布流程。当用户说"发布"、"发版"、"切版本"、"publish"、"release"，或 ROADMAP 已实施完成需要正式发布时使用。
---

# devops-release — 量潮 DevOps 包发布流程

## 流程

### 前置检查

```bash
qtcloud-devops status
```

统一 status 按生命周期顺序聚合输出：contract → source → plan → code → build → test → release。确认以下全部通过：

- 构建成功
- 测试全部通过
- 版本一致（配置文件版本与最新 tag 一致）
- 工作区干净

### 发布前审计

```bash
qtcloud-devops release audit -v <version>
```

审计 6+1 项：版本号格式、配置文件版本一致性、CHANGELOG、工作区状态、标签冲突、远程可达性、GitHub Release 同步。
全部通过后再发布。

### 预览版本号

先使用 `--dry-run` 预览自动检测的结果，AI 对其进行审议：

```bash
qtcloud-devops release publish --dry-run
# 📌 项目类型: code
# 📌 scope: Some("cli")
# 📦 最新标签: cli/v0.9.3-alpha.1
# 📝 提交数: 1
#    • feat: release publish -v optional with auto-detect
# 🧠 LLM 决策: feat 变更属于 minor，当前 alpha 阶段递增
# 🔮 建议版本: cli/v0.9.3-alpha.2
```

### 审议版本号

AI 审阅 `--dry-run` 的输出，对照[版本号规则](#版本号规则)逐一核查：

1. **项目类型** — code/docs 是否正确？
2. **scope** — 是否正确匹配了变更目录？
3. **是否发版** — 提交确实是逻辑改动，还是可以攒到下次？
4. **增量** — minor/patch 是否合理？`feat:` 不一定总是 minor，要看改动规模。
5. **预发布阶段** — alpha/beta/rc/正式 是否合适？已在预发布系列的同阶段递增是否正确，还是该晋级了？
6. **综合判断** — 版本号是否合理？不合理时与用户商议。

审议通过后，用 `-v` 显式指定版本号进行发布。不通过则与用户讨论调整。

### 发布

```bash
qtcloud-devops release publish -v <version> -y
```

`release publish` 内部自动完成：更新配置文件版本 → 生成 CHANGELOG → 创建 tag → 推送 → 创建 GitHub Release。

**所有步骤都是幂等的。** 超时或失败后直接重跑相同的 `publish` 命令，不会产生重复 tag 或 Release。

#### tag 或 Release 已存在（重新发布）

修复代码后，用 `--force` 强制重新发布：

```bash
qtcloud-devops release publish -v <version> -y -f
```

`--force` / `-f` 自动完成：

- 删除已存在的 GitHub Release
- 删除已存在的远端 tag
- 删除本地 tag
- 重新创建 tag → 推送 → 创建 Release

等效于手动执行以下步骤，但一次完成：

```bash
git push --delete origin cli/v0.8.3       # 删除远端 tag
gh release delete cli/v0.8.3 --yes        # 删除 GitHub Release
qtcloud-devops release publish -v cli/v0.8.3 -y
```

#### CI 未触发

监听 `release: [published]` 事件。如果 CI 未触发，用 `--force` 重新发布以触发：

```bash
qtcloud-devops release publish -v <version> -y -f
```

### 发布后检查

```bash
qtcloud-devops status
```

或只看发布状态：

```bash
qtcloud-devops release status
```

## 版本号规则

### 格式

- `vX.Y.Z` — 单包仓库版
- `scope/vX.Y.Z` — monorepo 正式版，如 `cli/v0.8.3`
- `scope/vX.Y.Z-prerelease` — 预发布版，如 `cli/v0.8.3-rc.1`

### 是否发版

- **发版** — 有用户可见的逻辑改动：`feat / fix / docs / refactor / test`。
  - `docs:` 是内容变更，在文档项目中就是交付物本身，算逻辑改动。
  - 代码项目中 `docs:` 通常与代码变更一起出现，同样算逻辑改动。
- **不发** — 仅 `chore / typo / CI 配置` 等非逻辑改动，攒到下次。

### minor 还是 patch

规则因项目类型而异，`detect` 会自动检测项目类型。

**代码项目：**

| 提交类型 | 增量 |
|---------|------|
| `feat:` | **minor**（Y+1） |
| `fix: / refactor: / test: / docs:` | **patch**（Z+1） |

**内容/文档项目：**

绝大多数变更是 patch。新增文档、更新内容、格式规范化、目录结构调整都是日常工作。

- 常规内容更新 → **patch**（Z+1）
- **minor** 仅限全新内容品类上线（如从零搭建一整套新手册），极少发生。
- 不确定时就 patch。

### 直发正式还是走预发布

| 项目类型 | 场景 | 发版方式 |
|---------|------|---------|
| 代码 | patch | **直发正式** |
| 代码 | minor | **走预发布**（-rc.1） |
| 代码 | 大版本周期（多个 minor） | **走预发布**（alpha→beta→rc） |
| 代码 | 正式版紧急 bug | **patch-rc.1** → 验证后转正式 |
| 代码 | 已在预发布系列 | **同阶段递增序号** |
| 文档 | 所有变更 | **直发正式**，不走预发布 |

**简单原则**：patch 直发正式，代码项目 minor 走预发布，文档项目全部直发正式。不确定时问用户。

### 预发布阶段

| 阶段 | 标签 | 起始时机 |
|------|------|---------|
| Alpha | `-alpha.N` | 大版本早期，功能未完成，API 不稳定 |
| Beta | `-beta.N` | 功能基本完成，API 冻结，供外部验证 |
| Release Candidate | `-rc.N` | 功能冻结，只修 bug，准备正式发版 |
| 正式版 | 无后缀 | 稳定发布 |

**晋级规则：**

- `alpha.k` → 功能够用时 → `beta.1`
- `beta.k` → 评审通过 → `rc.1`
- `rc.k` → 验证通过 → 去掉后缀发正式版
- 阶段切换时序号重置（`alpha.2` → `beta.1`）
- **正式发版（去掉后缀）由人类决定**，AI 不做

### major

**AI 不做 major bump。** 涉及 breaking change 或大版本升级，由人类指定版本号。

### 原则

**AI 做判断，人类做决定。** AI 负责常规判断（scope、发不发、minor/patch、预发布序号递增），重大拐点由人类控制（major 版本、正式发版去掉后缀）。不确定时问用户。
