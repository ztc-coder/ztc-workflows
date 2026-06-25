---
name: git-workflow
description: Git 工作流模式，包括分支策略、提交约定、合并与变基、冲突解决以及适用于各种规模团队的协作开发最佳实践。
origin: ECC
---

# Git 工作流模式

Git 版本控制、分支策略和协作开发的最佳实践。

## 何时激活

- 为新项目设置 Git 工作流
- 决定分支策略（GitFlow、主干开发、GitHub Flow）
- 编写提交信息和 PR 描述
- 解决合并冲突
- 管理发布版本和版本标签
- 帮助新团队成员熟悉 Git 实践

## 分支策略

### GitHub Flow（简单，推荐大多数情况使用）

最适合持续部署和中小型团队。

```
main（受保护，始终可部署）
  │
  ├── feature/user-auth      → PR → 合并到 main
  ├── feature/payment-flow   → PR → 合并到 main
  └── fix/login-bug          → PR → 合并到 main
```

**规则：**
- `main` 始终保持可部署状态
- 从 `main` 创建功能分支
- 准备好审查时创建 Pull Request
- 审查通过且 CI 通过后，合并到 `main`
- 合并后立即部署

### 主干开发（高效率团队）

最适合拥有强大 CI/CD 和功能开关的团队。

```
main（主干）
  │
  ├── 短期功能分支（最多 1-2 天）
  ├── 短期功能分支
  └── 短期功能分支
```

**规则：**
- 所有人都直接提交到 `main` 或使用非常短期的分支
- 使用功能开关隐藏未完成的工作
- CI 必须在合并前通过
- 每天多次部署

### GitFlow（复杂，以发布周期驱动）

最适合按计划发布和企业级项目。

```
main（生产发布）
  │
  └── develop（集成分支）
        │
        ├── feature/user-auth
        ├── feature/payment
        │
        ├── release/1.0.0    → 合并到 main 和 develop
        │
        └── hotfix/critical  → 合并到 main 和 develop
```

**规则：**
- `main` 仅包含生产就绪的代码
- `develop` 是集成分支
- 功能分支从 `develop` 创建，合并回 `develop`
- 发布分支从 `develop` 创建，合并到 `main` 和 `develop`
- 热修复分支从 `main` 创建，同时合并到 `main` 和 `develop`

### 如何选择

| 策略 | 团队规模 | 发布节奏 | 最适合 |
|----------|-----------|-----------------|----------|
| GitHub Flow | 不限 | 持续 | SaaS、web 应用、初创公司 |
| 主干开发 | 5+ 有经验 | 每天多次 | 高效率团队、功能开关 |
| GitFlow | 10+ | 按计划 | 企业级、受监管行业 |

## 提交信息

### Conventional Commits 格式

```
<type>(<scope>): <subject>

[可选正文]

[可选脚注]
```

### 类型

| 类型 | 用途 | 示例 |
|------|---------|---------|
| `feat` | 新功能 | `feat(auth): 添加 OAuth2 登录` |
| `fix` | 错误修复 | `fix(api): 处理用户端点中的空响应` |
| `docs` | 文档 | `docs(readme): 更新安装说明` |
| `style` | 格式化，不影响代码逻辑 | `style: 修正登录组件的缩进` |
| `refactor` | 代码重构 | `refactor(db): 将连接池提取为模块` |
| `test` | 添加/更新测试 | `test(auth): 添加令牌验证的单元测试` |
| `chore` | 维护任务 | `chore(deps): 更新依赖` |
| `perf` | 性能优化 | `perf(query): 为 users 表添加索引` |
| `ci` | CI/CD 变更 | `ci: 为测试工作流添加 PostgreSQL 服务` |
| `revert` | 回滚之前的提交 | `revert: 回滚 "feat(auth): 添加 OAuth2 登录"` |

### 好与坏的示例

```
# 不好：模糊、无上下文
git commit -m "修复了一些东西"
git commit -m "更新"
git commit -m "WIP"

# 好的：清晰、具体、解释原因
git commit -m "fix(api): 在 503 Service Unavailable 时重试请求

外部 API 在高峰时段偶尔返回 503 错误。
添加了指数退避重试逻辑，最多 3 次尝试。

Closes #123"
```

### 提交信息模板

在仓库根目录创建 `.gitmessage`：

```
# <type>(<scope>): <subject>
# # 类型：feat、fix、docs、style、refactor、test、chore、perf、ci、revert
# 范围：api、ui、db、auth 等
# 主题：祈使语气，不加句号，最多 50 个字符
#
# [可选正文] - 解释为什么，而不是做了什么
# [可选脚注] - 破坏性变更、关闭 issue
```

启用：`git config commit.template .gitmessage`

## 合并 vs 变基

### 合并（保留历史）

```bash
# 创建一个合并提交
git checkout main
git merge feature/user-auth

# 结果：
# *   合并提交
# |\
# | * 功能提交
# |/
# * main 提交
```

**适用场景：**
- 将功能分支合并到 `main`
- 想要保留完整历史记录
- 多人在分支上协作
- 分支已推送且其他人可能基于此分支继续工作

### 变基（线性历史）

```bash
# 将功能提交重写到目标分支上
git checkout feature/user-auth
git rebase main

# 结果：
# * 功能提交（已重写）
# * main 提交
```

**适用场景：**
- 用最新的 `main` 更新本地功能分支
- 想要线性、干净的历史记录
- 分支仅在本地（未推送）
- 你是分支上唯一的开发者

### 变基工作流

```bash
# 用最新的 main 更新功能分支（PR 之前）
git checkout feature/user-auth
git fetch origin
git rebase origin/main

# 修复所有冲突
# 测试应仍然通过

# 强制推送（仅当你是指定的唯一贡献者时使用）
git push --force-with-lease origin feature/user-auth
```

### 何时不应变基

```
# 绝不应对以下分支进行变基：
- 已推送到共享仓库的分支
- 其他人已基于其工作的分支
- 受保护的分支（main、develop）
- 已合并的分支

# 原因：变基会重写历史，破坏他人的工作
```

## Pull Request 工作流

### PR 标题格式

```
<type>(<scope>): <描述>

示例：
feat(auth): 为 enterprise 用户添加 SSO 支持
fix(api): 解决订单处理中的竞态条件
docs(api): 为 v2 端点添加 OpenAPI 规范
```

### PR 描述模板

```markdown
## 做了什么

简要描述此 PR 的功能。

## 为什么

解释动机和背景。

## 如何实现

值得关注的关键实现细节。

## 测试

- [ ] 单元测试已添加/更新
- [ ] 集成测试已添加/更新
- [ ] 已执行手动测试

## 截图（如适用）

UI 变更的前后对比截图。

## 检查清单

- [ ] 代码遵循项目风格指南
- [ ] 已完成自我审查
- [ ] 复杂逻辑已添加注释
- [ ] 文档已更新
- [ ] 未引入新警告
- [ ] 本地测试通过
- [ ] 已关联相关 issue

Closes #123
```

### 代码审查检查清单

**对审查者：**

- [ ] 代码是否解决了所述问题？
- [ ] 是否存在未处理的边界情况？
- [ ] 代码是否可读且可维护？
- [ ] 是否有足够的测试？
- [ ] 是否存在安全问题？
- [ ] 提交历史是否干净（如有需要已压缩）？

**对作者：**

- [ ] 提交审查前已完成自我审查
- [ ] CI 通过（测试、lint、类型检查）
- [ ] PR 大小合理（理想情况下 <500 行）
- [ ] 只关联一个功能/修复
- [ ] 描述清楚地解释了变更

## 冲突解决

### 识别冲突

```bash
# 合并前检查冲突
git checkout main
git merge feature/user-auth --no-commit --no-ff

# 如果有冲突，Git 会显示：
# CONFLICT（内容）：src/auth/login.ts 中的合并冲突
# 自动合并失败；修复冲突后提交结果。
```

### 解决冲突

```bash
# 查看冲突文件
git status

# 查看文件中的冲突标记
# <<<<<<< HEAD
# main 的内容
# =======
# 功能分支的内容
# >>>>>>> feature/user-auth

# 选项 1：手动解决
# 编辑文件，移除标记，保留正确内容

# 选项 2：使用合并工具
git mergetool

# 选项 3：接受一方的版本
git checkout --ours src/auth/login.ts    # 保留 main 版本
git checkout --theirs src/auth/login.ts  # 保留功能分支版本

# 解决后，暂存并提交
git add src/auth/login.ts
git commit
```

### 冲突预防策略

```bash
# 1. 保持功能分支小而短命
# 2. 频繁变基到 main
git checkout feature/user-auth
git fetch origin
git rebase origin/main

# 3. 与团队沟通涉及共享文件的修改
# 4. 使用功能开关替代长期分支
# 5. 及时审查并合并 PR
```

## 分支管理

### 命名约定

```
# 功能分支
feature/user-authentication
feature/JIRA-123-payment-integration

# 错误修复
fix/login-redirect-loop
fix/456-null-pointer-exception

# 热修复（生产问题）
hotfix/critical-security-patch
hotfix/database-connection-leak

# 发布版本
release/1.2.0
release/2024-01-hotfix

# 实验/POC
experiment/new-caching-strategy
poc/graphql-migration
```

### 分支清理

```bash
# 删除已合并的本地分支
git branch --merged main | grep -v "^\*\|main" | xargs -n 1 git branch -d

# 删除已删除的远程分支的远程跟踪引用
git fetch -p

# 删除本地分支
git branch -d feature/user-auth  # 安全删除（仅在已合并时）
git branch -D feature/user-auth  # 强制删除

# 删除远程分支
git push origin --delete feature/user-auth
```

### Stash 工作流

```bash
# 保存正在进行的工作
git stash push -m "WIP: 用户认证"

# 列出暂存内容
git stash list

# 应用最近的暂存
git stash pop

# 应用指定的暂存
git stash apply stash@{2}

# 删除暂存
git stash drop stash@{0}
```

## 发布管理

### 语义化版本

```
MAJOR.MINOR.PATCH

MAJOR：破坏性变更
MINOR：新功能，向后兼容
PATCH：错误修复，向后兼容

示例：
1.0.0 → 1.0.1（patch：错误修复）
1.0.1 → 1.1.0（minor：新功能）
1.1.0 → 2.0.0（major：破坏性变更）
```

### 创建发布版本

```bash
# 创建带注释的标签
git tag -a v1.2.0 -m "发布 v1.2.0

新功能：
- 添加用户认证
- 实现密码重置

修复：
- 解决登录重定向问题

破坏性变更：
- 无"

# 推送标签到远程
git push origin v1.2.0

# 列出标签
git tag -l

# 删除标签
git tag -d v1.2.0
git push origin --delete v1.2.0
```

### 变更日志生成

```bash
# 从提交历史生成变更日志
git log v1.1.0..v1.2.0 --oneline --no-merges

# 或使用 conventional-changelog
npx conventional-changelog -i CHANGELOG.md -s
```

## Git 配置

### 基本配置

```bash
# 用户身份
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 默认分支名称
git config --global init.defaultBranch main

# Pull 行为（变基而非合并）
git config --global pull.rebase true

# Push 行为（仅推送当前分支）
git config --global push.default current

# 自动纠正拼写错误
git config --global help.autocorrect 1

# 更好的 diff 算法
git config --global diff.algorithm histogram

# 彩色输出
git config --global color.ui auto
```

### 常用别名

```bash
# 添加到 ~/.gitconfig
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = log --oneline --graph --all
    amend = commit --amend --no-edit
    wip = commit -m "WIP"
    undo = reset --soft HEAD~1
    contributors = shortlog -sn
```

### Gitignore 模式

```gitignore
# 依赖
node_modules/
vendor/

# 构建输出
dist/
build/
*.o
*.exe

# 环境文件
.env
.env.local
.env.*.local

# IDE
.idea/
.vscode/
*.swp
*.swo

# 操作系统文件
.DS_Store
Thumbs.db

# 日志
*.log
logs/

# 测试覆盖率
coverage/

# 缓存
.cache/
*.tsbuildinfo
```

## 常用工作流

### 开始一个新功能

```bash
# 1. 更新 main 分支
git checkout main
git pull origin main

# 2. 创建功能分支
git checkout -b feature/user-auth

# 3. 进行修改并提交
git add .
git commit -m "feat(auth): 实现 OAuth2 登录"

# 4. 推送到远程
git push -u origin feature/user-auth

# 5. 在 GitHub/GitLab 上创建 Pull Request
```

### 用新变更更新 PR

```bash
# 1. 进行额外修改
git add .
git commit -m "feat(auth): 添加错误处理"

# 2. 推送更新
git push origin feature/user-auth
```

### 同步 Fork 与上游仓库

```bash
# 1. 添加上游远程仓库（仅需一次）
git remote add upstream https://github.com/original/repo.git

# 2. 获取上游更新
git fetch upstream

# 3. 将上游的 main 合并到你的 main
git checkout main
git merge upstream/main

# 4. 推送到你的 fork
git push origin main
```

### 撤销错误

```bash
# 撤销上次提交（保留更改）
git reset --soft HEAD~1

# 撤销上次提交（丢弃更改）
git reset --hard HEAD~1

# 撤销已推送到远程的上次提交
git revert HEAD
git push origin main

# 撤销特定文件的更改
git checkout HEAD -- path/to/file

# 修正上次提交信息
git commit --amend -m "新的信息"

# 将遗漏的文件添加到上次提交
git add forgotten-file
git commit --amend --no-edit
```

## Git Hooks

### Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 运行 lint
npm run lint || exit 1

# 运行测试
npm test || exit 1

# 检查敏感信息
if git diff --cached | grep -E '(password|api_key|secret)'; then
    echo "检测到可能的敏感信息。提交已中止。"
    exit 1
fi
```

### Pre-Push Hook

```bash
#!/bin/bash
# .git/hooks/pre-push

# 运行完整测试套件
npm run test:all || exit 1

# 检查 console.log 语句
if git diff origin/main | grep -E 'console\.log'; then
    echo "请在推送前移除 console.log 语句。"
    exit 1
fi
```

## 反模式

```
# 不好的做法：直接提交到 main
git checkout main
git commit -m "修复 bug"

# 好的做法：使用功能分支和 PR

# 不好的做法：提交敏感信息
git add .env  # 包含 API 密钥

# 好的做法：添加到 .gitignore，使用环境变量

# 不好的做法：超大 PR（1000+ 行）
# 好的做法：拆分为更小、更聚焦的 PR

# 不好的做法："Update" 类提交信息
git commit -m "update"
git commit -m "fix"

# 好的做法：描述清晰的提交信息
git commit -m "fix(auth): 解决登录后的重定向循环"

# 不好的做法：重写公共历史
git push --force origin main

# 好的做法：对公共分支使用 revert
git revert HEAD

# 不好的做法：长期存在的功能分支（数周/数月）
# 好的做法：保持分支短期（数天），频繁变基

# 不好的做法：提交生成文件
git add dist/
git add node_modules/

# 好的做法：添加到 .gitignore
```

## 快速参考

| 任务 | 命令 |
|------|---------|
| 创建分支 | `git checkout -b feature/name` |
| 切换分支 | `git checkout branch-name` |
| 删除分支 | `git branch -d branch-name` |
| 合并分支 | `git merge branch-name` |
| 变基分支 | `git rebase main` |
| 查看历史 | `git log --oneline --graph` |
| 查看更改 | `git diff` |
| 暂存更改 | `git add .` 或 `git add -p` |
| 提交 | `git commit -m "message"` |
| 推送 | `git push origin branch-name` |
| 拉取 | `git pull origin branch-name` |
| 暂存 | `git stash push -m "message"` |
| 撤销上次提交 | `git reset --soft HEAD~1` |
| 还原提交 | `git revert HEAD` |
