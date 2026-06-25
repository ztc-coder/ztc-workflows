---
name: find-skills
description: 当用户提出“how do I do X”“find a skill for X”“is there a skill that can...”之类问题，或表达扩展能力意愿时，帮助其发现并安装 agent skills。用户在寻找可能以可安装 skill 形式存在的功能时应使用此 skill。
---

# 查找 Skills

此 skill 帮助你从开放的 agent skills 生态中发现并安装 skills。

## 何时使用此 Skill

当用户：

- 询问“how do I do X”，且 X 可能是已有 skill 支持的常见任务
- 说“find a skill for X”或“is there a skill for X”
- 询问“can you do X”，且 X 属于专门能力
- 表达希望扩展 agent 能力
- 想搜索工具、模板或工作流
- 提到希望在某个特定领域（设计、测试、部署等）获得帮助

## 什么是 Skills CLI？

Skills CLI（`npx skills`）是开放 agent skills 生态的包管理器。Skills 是模块化包，通过专门知识、工作流和工具扩展 agent 能力。

**关键命令：**

- `npx skills find [query]` - 交互式或按关键词搜索 skills
- `npx skills add <package>` - 从 GitHub 或其他来源安装 skill
- `npx skills check` - 检查 skill 更新
- `npx skills update` - 更新所有已安装 skill

**浏览 skills：** https://skills.sh/

## 如何帮助用户查找 Skills

### 第 1 步：理解他们需要什么

当用户就某件事寻求帮助时，识别：

1. 领域（如 React、测试、设计、部署）
2. 具体任务（如编写测试、创建动画、审查 PR）
3. 这是否足够常见，以至于很可能已有对应 skill

### 第 2 步：搜索 Skills

使用相关查询运行查找命令：

```bash
npx skills find [query]
```

例如：

- 用户问“how do I make my React app faster?” → `npx skills find react performance`
- 用户问“can you help me with PR reviews?” → `npx skills find pr review`
- 用户问“I need to create a changelog” → `npx skills find changelog`

命令将返回如下结果：

```
Install with npx skills add <owner/repo@skill>

vercel-labs/agent-skills@vercel-react-best-practices
└ https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices
```

### 第 3 步：向用户展示选项

当你找到相关 skills 时，向用户展示：

1. Skill 名称及其作用
2. 他们可以运行的安装命令
3. 在 skills.sh 了解更多的链接

示例回复：

```
我找到一个可能有帮助的 skill！“vercel-react-best-practices” skill 提供
来自 Vercel Engineering 的 React 与 Next.js 性能优化指南。

安装命令：
npx skills add vercel-labs/agent-skills@vercel-react-best-practices

了解更多：https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices
```

### 第 4 步：提出代为安装

如果用户想继续，你可以为他们安装该 skill：

```bash
npx skills add <owner/repo@skill> -g -y
```

`-g` 标志表示全局安装（用户级），`-y` 跳过确认提示。

## 常见 Skill 分类

搜索时，可考虑这些常见分类：

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## 有效搜索技巧

1. **使用具体关键词**：`react testing` 比单独 `testing` 更好
2. **尝试替代词**：如果 `deploy` 不行，试试 `deployment` 或 `ci-cd`
3. **检查热门来源**：很多 skills 来自 `vercel-labs/agent-skills` 或 `ComposioHQ/awesome-claude-skills`

## 当找不到 Skills 时

如果不存在相关 skill：

1. 明确说明没有找到现有 skill
2. 提出直接使用你的通用能力帮助完成该任务
3. 建议用户使用 `npx skills init` 创建自己的 skill

示例：

```
我搜索了与 “xyz” 相关的 skills，但没有找到匹配项。
我仍然可以直接帮助你完成这个任务！要我继续吗？

如果这是你经常做的事，你可以创建自己的 skill：
npx skills init my-xyz-skill
```
