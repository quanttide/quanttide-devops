# AGENTS.md

## 项目定位

`quanttide-devops` 是量潮科技的 DevOps 基础设施仓库，聚合所有 DevOps 相关的仓库为子模块，并提供统一的 CLI 工具。

## 文档体系

```
docs/   → 给人读的（文章型文档）
  essay/         随笔
  gallery/       案例（做成了什么的实证陈列）
  handbook/      手册（生命周期视角：规划→编码→构建→测试→发布→部署→运维→监控）
  tutorial/      教程（主题视角：快速入门→Scrum→Git→流水线→制品→OpenAPI→设计开发→CI/CD→发布）
  bylaw/         章程（制度性约定）
  specification/ 标准（技术规范）
data/   → 当数据用的（结构化记录）
  archive/       归档站
  brochure/      宣传册
  context/       上下文
  history/       工作历史
  insight/       洞察
  intention/     意图
  journal/       日志
  report/        报告
  roadmap/       路线图
```

**核心划分原则：**
- handbook 按**流程**导航，回答"这个环节用什么"
- tutorial 按**主题**教学，回答"某个知识点怎么用"
- gallery 按**项目**陈列，回答"做成了什么、效果如何"
- data/ 下的内容是结构化记录，不是文章
- report/ 结构化记录，记录关键事件（如架构决策）

## 子模组

子模组列表和操作规范见 `CONTRIBUTING.md`。

## 初始化上下文

每次会话启动时，自动读取最近 3 天的 Human 日志和 Agent 日志（`data/journal/human/` 和 `data/journal/agent/`），以恢复工作上下文。

日志文件按日期命名（如 `2026-07-07.md`），按文件名排序取最近 3 份。

## 工作纪律

**必须使用 devops skill 或 `qtcloud-devops` CLI 自举管理 DevOps 活动。**

遵守已定义的范围划分，所有 DevOps 操作通过工具闭环，不绕开工具手动执行。

使用 `qtcloud-devops help` 学习命令，不在此处列举。

tool 不满足需求时，先扩展 tool，再通过 tool 执行。
