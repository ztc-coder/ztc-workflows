---
name: evolve
description: 分析 instincts（直觉）并建议或生成演化后的结构
command: true
---

# Evolve 命令

## 实现方式

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

或者当 `CLAUDE_PLUGIN_ROOT` 未设置时（手动安装场景）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

分析 instincts 并将相关项聚类为更高层级的结构：
- **Commands（命令）**：当 instincts 描述用户主动调用的动作时
- **Skills（技能）**：当 instincts 描述自动触发的行为时
- **Agents（代理）**：当 instincts 描述复杂的多步骤流程时

## 用法

```
/evolve                    # 分析所有 instincts 并建议演化方案
/evolve --generate         # 同时在 evolved/{skills,commands,agents} 下生成文件
```

## 演化规则

### → Command（用户主动调用）
当 instincts 描述用户会显式请求的动作时：
- 多个关于「当用户请求……时」的 instincts
- 带有类似「当创建新 X 时」trigger（触发器）的 instincts
- 遵循可重复序列的 instincts

示例：
- `new-table-step1`："when adding a database table, create migration"
- `new-table-step2`："when adding a database table, update schema"
- `new-table-step3`："when adding a database table, regenerate types"

→ 生成：**new-table** 命令

### → Skill（自动触发）
当 instincts 描述应当自动发生的行为时：
- 模式匹配 trigger
- 错误处理响应
- 代码风格强制执行

示例：
- `prefer-functional`："when writing functions, prefer functional style"
- `use-immutable`："when modifying state, use immutable patterns"
- `avoid-classes`："when designing modules, avoid class-based design"

→ 生成：`functional-patterns` skill

### → Agent（需要深度/隔离）
当 instincts 描述从隔离中获益的复杂多步骤流程时：
- 调试工作流
- 重构序列
- 研究任务

示例：
- `debug-step1`："when debugging, first check logs"
- `debug-step2`："when debugging, isolate the failing component"
- `debug-step3`："when debugging, create minimal reproduction"
- `debug-step4`："when debugging, verify fix with test"

→ 生成：**debugger** agent

## 执行步骤

1. 检测当前项目上下文
2. 读取项目级 + 全局 instincts（ID 冲突时项目级优先）
3. 按 trigger/domain（触发器/领域）模式对 instincts 分组
4. 识别：
   - Skill 候选项（包含 2 个以上 instincts 的 trigger 聚类）
   - Command 候选项（高 confidence（置信度）的工作流 instincts）
   - Agent 候选项（规模较大、高 confidence 的聚类）
5. 在适用时展示晋升候选项（project -> global）
6. 如果传入 `--generate`，则将文件写入：
   - 项目 scope（范围）：`~/.claude/homunculus/projects/<project-id>/evolved/`
   - 全局回退：`~/.claude/homunculus/evolved/`

## 输出格式

```
============================================================
  EVOLVE ANALYSIS - 12 instincts
  Project: my-app (a1b2c3d4e5f6)
  Project-scoped: 8 | Global: 4
============================================================

High confidence instincts (>=80%): 5

## SKILL CANDIDATES
1. Cluster: "adding tests"
   Instincts: 3
   Avg confidence: 82%
   Domains: testing
   Scopes: project

## COMMAND CANDIDATES (2)
  /adding-tests
    From: test-first-workflow [project]
    Confidence: 84%

## AGENT CANDIDATES (1)
  adding-tests-agent
    Covers 3 instincts
    Avg confidence: 82%
```

## 标志位

- `--generate`：除分析输出外，还生成演化后的文件

## 生成文件格式

### Command
```markdown
---
name: new-table
description: 创建一个新的数据库表，包含迁移、schema 更新和类型生成
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table Command

[基于聚类后的 instincts 生成的内容]

## Steps
1. ...
2. ...
```

### Skill
```markdown
---
name: functional-patterns
description: 强制执行函数式编程模式
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns Skill

[基于聚类后的 instincts 生成的内容]
```

### Agent
```markdown
---
name: debugger
description: 系统化的调试代理
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger Agent

[基于聚类后的 instincts 生成的内容]
```
