---
description: 代码审查 — 本地未提交的改动或 GitHub PR（传入 PR 编号/URL 进入 PR 模式）
argument-hint: [pr-number | pr-url | 留空则为本地审查]
---

# 代码审查

> PR 审查模式改编自 Wirasm 的 PRPs-agentic-eng，属于 PRP 工作流系列。

**输入**：$ARGUMENTS

---

## 模式选择

如果 `$ARGUMENTS` 包含 PR 编号、PR URL 或 `--pr`：
→ 跳转到下方的 **PR 审查模式**。

否则：
→ 使用**本地审查模式**。

---

## 本地审查模式

对未提交变更进行全面的安全和质量审查。

### 阶段 1 — 收集

```bash
git diff --name-only HEAD
```

若无变更文件，停止：「无内容可审查。」

### 阶段 2 — 审查

完整读取每个变更文件。检查：

**安全问题（CRITICAL）：**
- 硬编码凭据、API keys、tokens
- SQL injection 漏洞
- XSS 漏洞
- 缺少输入验证
- 不安全依赖
- 路径遍历风险

**代码质量（HIGH）：**
- 函数 > 50 行
- 文件 > 800 行
- 嵌套深度 > 4 层
- 缺少错误处理
- console.log 语句
- TODO/FIXME 注释
- 公共 API 缺少 JSDoc

**最佳实践（MEDIUM）：**
- 变更模式（改用不可变方式）
- 代码/注释中使用 emoji
- 新代码缺少测试
- 无障碍问题（a11y）

### 阶段 3 — 报告

生成包含以下内容的报告：
- 严重程度：CRITICAL、HIGH、MEDIUM、LOW
- 文件位置和行号
- 问题描述
- 建议修复方案

发现 CRITICAL 或 HIGH 问题时阻断提交。
绝不批准含安全漏洞的代码。

---

## PR 审查模式

全面的 GitHub PR 审查 — 获取 diff、读取完整文件、运行验证、发布审查。

### 阶段 1 — 获取

```bash
# 获取 PR 详情
gh pr view <NUMBER> --json title,body,author,baseRefName,headRefName,files

# 获取 diff
gh pr diff <NUMBER>
```

### 阶段 2 — 分析

对每个变更文件：
1. 读取完整文件（不只是 diff）
2. 理解上下文和目的
3. 应用审查清单

### 阶段 3 — 验证

运行验证检查：

```bash
# TypeScript 类型检查
npx tsc --noEmit 2>&1 | head -20

# 代码检查
npm run lint 2>&1 | head -20

# 测试
npm test -- --passWithNoTests 2>&1 | tail -20
```

### 阶段 4 — 安全扫描

```bash
# 检查密钥
grep -rn "sk-\|api_key\|password\|secret" --include="*.ts" --include="*.js" . 2>/dev/null | grep -v ".env\|test\|spec\|example" | head -10

# 检查 SQL injection 模式
grep -rn "query.*\${" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
```

### 阶段 5 — 汇总

在 `.claude/PRPs/reviews/pr-<NUMBER>-review.md` 创建审查摘要：

```markdown
# PR 审查：#<NUMBER> - <TITLE>

## 摘要
[2-3 句话概述]

## 决定：<APPROVE|REQUEST_CHANGES|BLOCK>

## 发现的问题
### CRITICAL
- [问题描述]（file:line）

### HIGH
- [问题描述]（file:line）

### MEDIUM
- [问题描述]（file:line）

## 验证结果
- TypeScript：PASS/FAIL
- Lint：PASS/FAIL
- 测试：PASS/FAIL（X/Y）

## 积极观察
- [做得好的地方]
```

### 阶段 6 — 发布审查

将审查发布到 GitHub：

```bash
# 批准
gh pr review <NUMBER> --approve --body "$(cat .claude/PRPs/reviews/pr-<NUMBER>-review.md)"

# 请求变更
gh pr review <NUMBER> --request-changes --body "$(cat .claude/PRPs/reviews/pr-<NUMBER>-review.md)"

# 仅评论
gh pr review <NUMBER> --comment --body "$(cat .claude/PRPs/reviews/pr-<NUMBER>-review.md)"
```

### 阶段 7 — 行内评论

对特定行级反馈，发布行内评论：

```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/comments" \
  -f body="<comment text>" \
  -f path="<file-path>" \
  -F line=<line-number> \
  -f side="RIGHT" \
  -f commit_id="$(gh pr view <NUMBER> --json headRefOid --jq .headRefOid)"
```

或者，一次性发布含多条行内评论的单个审查：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/reviews" \
  -f event="COMMENT" \
  -f body="<overall summary>" \
  --input comments.json  # [{"path": "file", "line": N, "body": "comment"}, ...]
```

### 阶段 8 — 输出

向用户报告：

```
PR #<NUMBER>: <TITLE>
决定：<APPROVE|REQUEST_CHANGES|BLOCK>

问题：<critical_count> 个 critical，<high_count> 个 high，<medium_count> 个 medium，<low_count> 个 low
验证：<pass_count>/<total_count> 项检查通过

产出物：
  审查：.claude/PRPs/reviews/pr-<NUMBER>-review.md
  GitHub：<PR URL>

后续步骤：
  - <基于决定的上下文建议>
```

---

## 边界情况

- **无 `gh` CLI**：回退到仅本地审查（读取 diff，跳过 GitHub 发布）。警告用户。
- **分支已分叉**：建议在审查前执行 `git fetch origin && git rebase origin/<base>`。
- **大型 PR（>50 个文件）**：警告审查范围。优先处理源码变更，然后是测试，最后是配置/文档。
