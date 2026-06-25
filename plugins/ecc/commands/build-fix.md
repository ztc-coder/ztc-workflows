---
name: build-fix
description: 最小化改动修复构建和类型错误。调用 build-error-resolver agent 执行。
command: true
allowed_tools: ["Bash", "Read", "Write", "Edit", "Grep", "Glob"]
argument-hint: ""
---

# 构建与修复

以最小、安全的改动逐步修复构建和类型错误。

## 第一步：检测构建系统

识别项目的构建工具并运行构建：

| 标识文件 | 构建命令 |
|-----------|---------------|
| `package.json` 含 `build` 脚本 | `npm run build` 或 `pnpm build` |
| `tsconfig.json`（仅 TypeScript） | `npx tsc --noEmit` |
| `Cargo.toml` | `cargo build 2>&1` |
| `pom.xml` | `mvn compile` |
| `build.gradle` | `./gradlew compileJava` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `python -m compileall -q .` 或 `mypy .` |

## 第二步：解析并分组错误

1. 运行构建命令并捕获 stderr
2. 按文件路径对错误分组
3. 按依赖顺序排序（先修复 import/类型错误，再处理逻辑错误）
4. 统计错误总数以追踪进度

## 第三步：修复循环（每次修复一个错误）

对每个错误：

1. **读取文件** — 使用 Read 工具查看错误上下文（错误前后各 10 行）
2. **诊断** — 找出根本原因（缺少 import、类型错误、语法错误）
3. **最小化修复** — 使用 Edit 工具做最小改动来解决错误
4. **重新运行构建** — 验证错误已消除且未引入新错误
5. **继续下一个** — 处理剩余错误

## 第四步：防护措施

遇到以下情况时停止并询问用户：
- 修复引入的错误**多于解决的错误**
- **同一错误在 3 次尝试后仍然存在**（可能是更深层的问题）
- 修复需要**架构层面的改动**（而非单纯的构建修复）
- 构建错误源于**缺少依赖**（需要 `npm install`、`cargo add` 等）

## 第五步：总结

展示结果：
- 已修复的错误（含文件路径）
- 剩余错误（如有）
- 新引入的错误（应为零）
- 未解决问题的后续建议

## 恢复策略

| 情况 | 处理方式 |
|-----------|--------|
| 缺少模块/import | 检查包是否已安装；建议安装命令 |
| 类型不匹配 | 读取两个类型定义；修复范围更窄的类型 |
| 循环依赖 | 用 import 图识别循环；建议提取公共模块 |
| 版本冲突 | 检查 `package.json` / `Cargo.toml` 中的版本约束 |
| 构建工具配置错误 | 读取配置文件；与可用的默认配置对比 |

每次只修复一个错误以保证安全。优先使用最小 diff，避免重构。
