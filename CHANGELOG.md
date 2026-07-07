# CHANGELOG



## [0.2.3] - 2026-07-07

### Added

- 新增 data/intention 子模块，用于工作意图管理  
- 新增 code-review skill，支持 qtcloud-code review  

### Changed

- 更新多个子模块（qtcloud-devops、docs/handbook、tutorial、journal、qtcloud-code、toolkit）至最新版本，优化路线图、计划文档及审计工具  
- 重构 plan edit 为 plan doctor，并更新相关技能文档  
- 从 CLI 和 toolkit 路线图中移除 unwrap 检查
## [0.2.2] - 2026-07-06

### Added

1. 新增 `qtcloud-code` 子模块，纳入应用层管理。  
2. 增加 `devops`、`devops-build`、`devops-code` 等多个 SKILL 文件，完善 CLI 生命周期、评审标准及架构原则。  
3. 在文档中新增 `report/` 原则和自我引导规范。

### Changed

1. 更新多个子模块（`qtcloud-devops`、`toolkit`、`qtadmin`、`lab`、`data/report` 等）至较新版本，同步指针并移除冗余内容。  
2. 重写并调整 `devops` 技能文档：整合命令路由与评审标准，调整流程顺序（`doctor`→`clean`），区分 `status` 与 `audit`。  
3. 统一文档标题：将“工具箱”重命名为“第二大脑”，`toolkit` 重命名为“工具箱”。  
4. 重构 `AGENTS.md`：移动子模块列表到 `CONTRIBUTING`，重写自举规则，替换命令列表为帮助提示。  
5. 将 CLI 技能同步至 v0.10.0-beta.2（`doctor`→`source`，新增 `audit`、`clean`，`doctor`→`edit`）。

### Fixed

1. 修复 `qtcloud-devops` 子模块的 CI 编译错误。  
2. 修复 `qtcloud-devops` 子模块的 registry 依赖问题。  
3. 修复 `qtcloud-devops` 子模块中 `python.rs` 的相关错误。  
4. 修正 `AGENTS.md` 中 `report/` 的描述。

### Removed

1. 移除 `ROADMAP.md` 及依赖真实 ROADMAP 的集成测试。  
2. 清理 `qtcloud-devops` 子模块中的热点文档（hotspots）。  
3. 移除已推进到平台的重复代码（如 `examples/default` 中的 `detect` 等）。
## [0.2.1] - 2026-07-05

### Added
- 新增多项实验与功能：git2 vs gix 对比实验、覆盖率工具性能对比、Docker proxy 支持、test run 容器执行、项目类型检测与 SKILL 规则对齐、ROADMAP.md 和 plan-command 蓝图、source::changelog 模块等。
- 新增多个子模块：quanttide-context-of-devops、docs/gallery（quanttide-gallery-of-devops）、data/history、data/bylaw 等。
- 新增多项文档：Docker 容器测试与代理指南、版本规则原则（AI 做判断，人类做决定）、devops-release 技能参考、devops-test 技能等。

### Changed
- 重构：toolkit 子模块路径变动（packages/toolkit → packages/quanttide-devops-toolkit）、model.rs 拆分为独立模块、devops-release 技能重写为 Agent 决策框架。
- 大量更新子模块引用至最新版本（涉及 qtcloud-devops、handbook、tutorial、lab、roadmap、by-law、specification、essay、report 等多个子模块）。
- 文档体系更新：AGENTS.md 同步文档树与子模块清单；发布流程改为先 `--dry-run` 预览再审议。
- 标准规范更新：契约文件规范重写（四维架构）、版本号格式标准化（统一去掉 v 前缀）。

### Fixed
- 修复 check_ci workflow 约定及 number 解析问题。
- 修复 pyproject.toml 版本同步、contract.yaml 移至仓库根目录、build status 保持只读等配置问题。
- 修复代码缺陷：python.rs 修复、P0 缺陷修复、覆盖率阈值解析修复（直接解析 LCOV）、CI 工作流修复等。

### Removed
- 移除冗余代理参数、删除代理配置章节、删除脚本和资源限制章节。
- 删除多余的预发布章节及复杂恢复流程（publish 幂等后无需复杂恢复）。
- 移除 git2 依赖，全部替换为 git CLI（子模块）。
- 删除 test run，改用手动执行各语言测试命令。

## [0.2.0] - 2026-05-25

### Added
- 新增 quanttide-report-of-devops 子模块（docs/report）
- 新增 quanttide-brochure-of-devops 子模块（docs/brochure）
- 新增 quanttide-journal-of-devops 子模块（docs/journal）

### Changed
- 重命名 examples/default 子模块为 quanttide-laboratory-of-devops
- 移除 examples/code 子模块

### Docs
- 更新 README 子模块列表

## [0.1.4] - 2026-05-24

### Added
- 添加 quanttide-report-of-devops 子模块（docs/report）

## [0.1.3] - 2026-05-24

### Added
- 添加 quanttide-essay-of-devops 子模块（docs/essay）

## [0.1.2] - 2026-05-23

### Added
- 添加 quanttide-example-of-devops 子模块（examples/default）

## [0.1.1] - 2026-05-23

### Added
- 添加 quanttide-roadmap-of-devops 子模块

### Docs
- 添加 release 子命令设计文档
- 添加 release 生命周期管理蓝图

## [0.1.0] - 2026-05-22

### Added
- 项目初始化
- 添加 qtcloud-devops 应用子模块
- 添加 devops-toolkit 工具包子模块
