---
name: instinct-status
description: 显示已学到的 instinct（项目 + 全局）及其 confidence
command: true
---

# Instinct Status 命令

显示当前项目以及全局的已学到 instinct，按 domain（领域）分组展示。

## 实现

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

或者，如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装），使用：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 用法

```
/instinct-status
```

## 执行步骤

1. 检测当前项目上下文（git remote/路径哈希）
2. 从 `~/.claude/homunculus/projects/<project-id>/instincts/` 读取项目 instinct
3. 从 `~/.claude/homunculus/instincts/` 读取全局 instinct
4. 按优先级规则合并（当 ID 冲突时，项目 instinct 覆盖全局 instinct）
5. 按 domain 分组展示，附带 confidence 进度条与 observation（观察）统计

## 输出格式

```
============================================================
  INSTINCT STATUS - 12 total
============================================================

  Project: my-app (a1b2c3d4e5f6)
  Project instincts: 8
  Global instincts:  4

## PROJECT-SCOPED (my-app)
  ### WORKFLOW (3)
    ███████░░░  70%  grep-before-edit [project]
              trigger: when modifying code

## GLOBAL (apply to all projects)
  ### SECURITY (2)
    █████████░  85%  validate-user-input [global]
              trigger: when handling user input
```
