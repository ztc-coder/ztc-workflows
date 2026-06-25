---
description: 重述需求、评估风险并创建逐步实现计划。在触碰任何代码前必须等待用户确认。
---

# Plan Command

此命令调用 **planner** agent，在编写任何代码前创建全面的实现计划。

## 此命令的功能

1. **重述需求** — 明确需要构建什么
2. **识别风险** — 暴露潜在问题和阻碍
3. **创建步骤计划** — 将实现拆分为阶段
4. **等待确认** — 必须获得用户批准后才能继续

## 使用时机

在以下情况使用 `/plan`：
- 开始新功能
- 进行重大架构变更
- 处理复杂重构
- 将影响多个文件/组件
- 需求不清晰或存在歧义

## 工作原理

planner agent 将：

1. **分析请求**并用清晰的语言重述需求
2. **拆分为阶段**，包含具体、可执行的步骤
3. **识别依赖**关系
4. **评估风险**和潜在阻碍
5. **估算复杂度**（高/中/低）
6. **呈现计划**并等待你的明确确认

## 使用示例

```
用户：/plan 我需要在市场解决时添加实时通知

Agent (planner):
# 实现计划：实时市场解决通知

## 需求重述
- 当用户关注的市场解决时向其发送通知
- 支持多种通知渠道（应用内、邮件、webhook）
- 确保通知可靠送达
- 包含市场结果和用户的持仓结果

## 实现阶段

### 阶段 1：数据库 Schema
- 添加 notifications 表，字段：id、user_id、market_id、type、status、created_at
- 添加 user_notification_preferences 表用于渠道偏好
- 在 user_id 和 market_id 上创建索引以提升性能

### 阶段 2：通知服务
- 在 lib/notifications.ts 创建通知服务
- 使用 BullMQ/Redis 实现通知队列
- 添加失败投递的重试逻辑
- 创建通知模板

### 阶段 3：集成点
- 接入市场解决逻辑（当状态变为"resolved"时）
- 查询市场中有持仓的所有用户
- 为每个用户入队通知

### 阶段 4：前端组件
- 在 header 创建 NotificationBell 组件
- 添加 NotificationList 弹窗
- 通过 Supabase 订阅实现实时更新
- 添加通知偏好页面

## 依赖
- Redis（用于队列）
- 邮件服务（SendGrid/Resend）
- Supabase 实时订阅

## 风险
- HIGH：邮件送达率（需要 SPF/DKIM）
- MEDIUM：每个市场 1000+ 用户时的性能
- MEDIUM：市场频繁解决时的通知垃圾
- LOW：实时订阅开销

## 预估复杂度：MEDIUM
- 后端：4-6 小时
- 前端：3-4 小时
- 测试：2-3 小时
- 合计：9-13 小时

**等待确认**：是否继续此计划？（yes/no/modify）
```

## 重要说明

**关键**：planner agent 在你明确用"yes"、"proceed"或类似肯定回复确认计划前，**不会**编写任何代码。

如需修改，请回复：
- "modify: [你的修改]"
- "different approach: [替代方案]"
- "skip phase 2 and do phase 3 first"

## 与其他命令的集成

规划完成后：
- 使用 `tdd-workflow` skill 配合测试驱动开发实现
- 发生构建错误时使用 `/build-fix`
- 使用 `/code-review` 审查完成的实现

<!-- > **需要更深入的规划？** 使用 `/prp-plan` 进行含 PRD 集成、代码库分析和模式提取的产出物规划。使用 `/prp-implement` 配合严格验证循环执行这些计划。 -->
## 相关 Agent

此命令调用 ecc 提供的 `planner` agent。

手动安装时，源文件位于：
`agents/planner.md`
