# Agent 编排

## 立即使用代理

无需用户额外提示。任务分级和风险处理遵循 [development-workflow.md](./development-workflow.md)：

1. 中型复杂功能或重构 → 使用 **planner**；大型或高风险任务必须使用。
2. 中型行为变更 → 使用 **code-reviewer**；大型或高风险任务必须使用。
3. 新功能、bug 修复或复杂逻辑 → 使用 **tdd-guide**。
4. TypeScript/JavaScript 代码 → 使用 **typescript-reviewer**。
5. 死代码清理 → 使用 **refactor-cleaner**。
6. 代码复杂度降低 → 使用 **code-simplifier**。
7. 独立任务 → 并行启动多个 agent。

## 并行任务执行

对独立操作始终使用并行 Task 执行：

```markdown
# 好：并行执行
同时启动 3 个代理：
1. 代理 1：认证模块安全分析
2. 代理 2：缓存系统性能审查
3. 代理 3：工具类型审查

# 坏：不必要的顺序
先代理 1，然后代理 2，然后代理 3
```

## 多视角分析

对于复杂问题，使用分角色子代理：
- 事实审查者
- 高级工程师
- 安全专家
- 一致性审查者
- 冗余检查者
