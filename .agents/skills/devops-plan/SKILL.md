---
name: devops-plan
description: 量潮 DevOps 规划管理。当用户说"规划"、"roadmap"、"进度"、"清理规划"、"修复规划"、"plan"、"ROADMAP"时使用。
---

# devops-plan — 量潮 DevOps 规划管理

> ROADMAP.md 的全生命周期管理：诊断 → 修复 → 清理 → 写入。

## 命令

| 命令 | 用途 |
|------|------|
| `qtcloud-devops plan status [scope]` | 查看 scope 规划进度 |
| `qtcloud-devops plan doctor [scope]` | 修复 scope 格式问题（LLM 修复 + 规则修复） |
| `qtcloud-devops plan clean [scope]` | 删除 scope 已完成条目（级联清理空分类和空版本） |

## ROADMAP.md 格式

标准格式（Keep a Changelog 变体）：

```markdown
# ROADMAP

## [0.2.0] — 进行中

### Added
- [x] 已完成功能
- [ ] 待办功能

### Changed
- [ ] 待重构

## [0.1.0] — 已发布

### Added
- [x] 初始功能
```

### 规则

| 要素 | 要求 | 示例 |
|------|------|------|
| 文档标题 | 首行必须 `# ROADMAP` | `# ROADMAP` |
| 版本头 | `## [X.Y.Z]` 可选 `— 状态` | `## [0.2.0] — 进行中` |
| 分类 | 标准大小写 | `### Added` / `### Changed` / `### Fixed` / `### Removed` / `### Deprecated` / `### Security` |
| 条目 | `- [x]` 已完成 / `- [ ]` 待办 | `- [x] 功能已完成` |

## 流程

### 1. 查看进度

```bash
qtcloud-devops plan status [scope]
```

输出各版本完成数/总数/百分比 + 总计进度。scope 省略时按当前工作目录自动匹配。

**判断后续操作：**
- 进度正常 → 跳过 doctor，直接到 clean 或 write
- 显示"未找到标准规划条目"或数据异常 → 进入 doctor

### 2. 修复格式（仅在 status 异常时）

```bash
qtcloud-devops plan doctor [scope]
```

两阶段修复：
1. **LLM 修复**：理解非标准格式并转换为标准格式（LLM 已配置时）
2. **规则校验**：v 前缀、分类大小写、checkbox 格式（双重保障）

doctor 后再跑一次 `plan status` 确认修复结果。

### 3. 清理已完成条目

```bash
qtcloud-devops plan clean [scope]
```

- 删除所有 `- [x]` 行
- 级联清理空分类（`### Added` 无内容时删除）
- 级联清理空版本（`## [X.Y.Z]` 无内容时删除）
- 自动 git commit

### 4. 写入新规划（手动编辑）

> 没有 CLI 命令用于创建规划条目。如需添加新版本或新条目，直接编辑 ROADMAP.md。

**添加新版本：**

在文件顶部（最新在前）插入：

```markdown
## [0.3.0] — 规划中

### Added
- [ ] 新功能描述
```

**添加待办条目到已有版本：**

在对应分类下追加 `- [ ] 描述` 行：

```markdown
### Changed
- [ ] 现有待办
- [ ] 新待办
```

**提交变更：**

```bash
git add ROADMAP.md
git commit -m "docs: 更新 ROADMAP 规划"
```

### 完整工作流示例

```bash
# 1. 查看当前进度
qtcloud-devops plan status

# 输出异常 → 2. 修复格式
qtcloud-devops plan doctor
qtcloud-devops plan status   # 确认修复结果

# 3. 清理已完成条目
qtcloud-devops plan clean

# 4. 手动编辑 ROADMAP.md 添加新规划
# vim ROADMAP.md ...

# git commit
git add ROADMAP.md
git commit -m "docs: 更新 ROADMAP 规划至 v0.3.0"
```

## scope 解析

1. **显式指定**：`plan status cli` → scope 名称匹配 `ROADMAP.md`
2. **自动检测**：省略时按当前工作目录匹配 contract scope
3. **回退**：无匹配时使用 `ROADMAP.md`

## 与 toolkit 的关系

| 功能 | CLI 实现 | toolkit 模块 |
|------|---------|------------|
| 解析 ROADMAP | `plan.rs` 行解析 | `source::roadmap::Roadmap` |
| 进度统计 | `VersionProgress` | `RoadmapVersion.percent()` |
| 格式验证 | `doctor_roadmap` | `Roadmap.validate()` |

CLI 的 `plan status` 可委托给 toolkit 的 `Roadmap` 解析；`plan clean/doctor` 的修复逻辑保持在 CLI 层。

## 常见问题

### plan status 显示"未找到标准规划条目"

ROADMAP.md 使用了非标准格式（如无 `# ROADMAP` 标题或自定义版本头）。
先 `plan doctor` 通过 LLM 自动转换，再 `plan status` 确认。

### plan clean 删多了怎么办

`plan clean` 会 git commit，可以通过 `git revert` 恢复。
建议在 clean 前先 `plan status` 确认进度正确。

### 不想每次传 scope 参数

在仓库根或各 scope 目录的 `ROADMAP.md` 中按标准格式编写，
`plan status` 省略 scope 时会自动匹配当前目录所在 scope。
