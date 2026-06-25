---
name: promote
description: 将 project（项目）范围的 instinct 提升至 global（全局）范围
command: true
---

# Promote 命令

在 continuous-learning-v2 中，将 instinct 从 project scope（项目范围）提升到 global scope（全局范围）。

## 实现

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" promote [instinct-id] [--force] [--dry-run]
```

或者如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装时）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py promote [instinct-id] [--force] [--dry-run]
```

## 用法

```bash
/promote                      # 自动检测可提升的候选项
/promote --dry-run            # 预览自动提升的候选项
/promote --force              # 无需确认即提升所有合格的候选项
/promote grep-before-edit     # 从当前 project 中提升某个特定的 instinct
```

## 操作步骤

1. 检测当前 project
2. 如果提供了 `instinct-id`，则只提升该 instinct（前提是它存在于当前 project 中）
3. 否则，查找满足以下条件的跨项目候选：
   - 至少出现在 2 个 project 中
   - 满足 confidence（置信度）阈值
4. 将被提升的 instinct 写入 `~/.claude/homunculus/instincts/personal/`，并标记 `scope: global`
