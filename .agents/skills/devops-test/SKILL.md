---
name: devops-test
description: 量潮 DevOps 测试流程。当用户说"测试"、"跑测试"、"覆盖率"、"coverage"、"test"、"测试阶段"、"CI 失败"时使用。
---

# devops-test — 量潮 DevOps 测试流程

## 测试命令

| 命令 | 用途 |
|------|------|
| `qtcloud-devops test status` | 查看测试状态（通过数、失败数、覆盖率） |
| `qtcloud-devops test run` | 运行测试 + 生成覆盖率报告 |
| `qtcloud-devops build status` | 查看构建状态（编译、依赖、版本一致性） |
| `qtcloud-devops doctor status` | 检查开发工具链（git/gh/cargo/python 等） |
| `qtcloud-devops status` | 聚合概览：contract → doctor → plan → code → build → test → release |

## 流程

### 1. 诊断环境

```bash
qtcloud-devops doctor status
```

确认项目用到的语言对应的工具链全部可用（git、gh、cargo、python、go、flutter、node 等按需显示）。

### 2. 运行测试

```bash
qtcloud-devops test run
```

自动按 scope 执行对应语言的测试命令：

| 语言 | 测试命令 | 覆盖率命令 |
|------|---------|-----------|
| Rust | `cargo test` | `cargo llvm-cov --lcov` |
| Python | `python -m pytest` | `coverage xml` |
| Go | `go test ./...` | `go tool cover` |
| Dart/Flutter | `flutter test` | `flutter test --coverage` |
| TypeScript | `npm test` | `npx nyc --reporter=lcov` |

覆盖率报告生成后，`test status` 即可读取并显示百分比。

### 3. 检查结果

```bash
qtcloud-devops test status
```

确认以下全部通过：

- 测试全部通过（0 failed）
- 覆盖率 ≥ 80%（默认阈值，可在 contract.yaml 中调整）
- 构建通过（`cargo check` / `uv check` / `go vet` 等）

### 4. 写入复盘

如果 CI 失败或测试发现问题，将失败原因、修复方式、后续预防措施写入 `data/report/`。

## 覆盖率阈值

默认 80%，定义在 `default_threshold()` 中。可在 contract.yaml 按 scope 覆盖：

```yaml
stages:
  test:
    threshold: 90

scopes:
  cli:
    test_threshold: 85  # scope 级覆盖
```

`test status` 输出中会显示阈值和是否达标：

```text
覆盖率: 88.3% ✅（阈值 80%）
```

## 测试方法论

### 分层测试

```
CLI 集成测试（tests/cli.rs）     — 通过真实二进制运行
库集成测试（tests/release.rs）   — 调用库函数 API
单元测试（src/**/mod.rs）        — 纯函数和格式化分支
```

### 可测试性

输出函数使用 `_to(writer)` 模式，不直接 `println!`：

```rust
// 可测试
pub fn status_to(writer: &mut impl Write) -> io::Result<()> {
    writeln!(writer, "结果: {}", data)?;
}
```

### 外部命令处理

映射函数（语言→命令）是纯函数，值得单元测试。执行函数（实际跑命令）不单独测，让集成测试覆盖。

### 覆盖率目标

| 区间 | 策略 |
|------|------|
| < 80% | 补基本测试 |
| 80-90% | 重构提可测试性（println → writeln） |
| 90-95% | 加集成测试覆盖 main.rs 和 I/O |
| > 95% | 接受，不继续投入 |

## 常见问题

### 覆盖率低于阈值

```bash
# 本地生成覆盖率报告查看详情
cargo llvm-cov --lcov --output-path target/coverage/lcov.info
# 检查哪些模块未覆盖
# 补测试后重新运行
qtcloud-devops test run
qtcloud-devops test status
```

### 测试失败

`test run` 失败时会中断，不会继续生成覆盖率。先修复测试，再重新运行。

### CI 测试失败

CI 中 `test-rust` workflow 监听 `release: [published]` 事件自动触发。如果失败：

1. 本地复现：`cargo test`
2. 查看失败原因
3. 修复后 `git push`，推送会自动触发新的 CI 运行
4. CI 通过后继续发布流程
