---
name: tdd-workflow
description: 编写新功能、修复 bug 或重构代码时使用此 skill。强制执行测试驱动开发，覆盖率 80%+，包含单元测试、集成测试和 E2E 测试。
origin: ECC
---

# 测试驱动开发工作流

此 skill 确保所有代码开发遵循 TDD 原则，并具备全面的测试覆盖率。

## 激活时机

- 编写新功能或新特性
- 修复 bug 或问题
- 重构现有代码
- 添加 API 端点
- 创建新组件

## 核心原则

### 1. 代码前先写测试
始终先写测试，再编写代码使测试通过。

### 2. 覆盖率要求
- 最低 80% 覆盖率（单元测试 + 集成测试 + E2E）
- 覆盖所有边界情况
- 测试错误场景
- 验证边界条件

### 3. 测试类型

#### 单元测试
- 独立函数和工具函数
- 组件逻辑
- 纯函数
- 辅助函数和工具

#### 集成测试
- API 端点
- 数据库操作
- 服务交互
- 外部 API 调用

#### E2E 测试（Playwright）
- 关键用户流程
- 完整工作流
- 浏览器自动化
- UI 交互

### 4. Git 检查点
- 若仓库使用 Git，在每个 TDD 阶段后创建检查点提交
- 在工作流完成前不要压缩或重写这些检查点提交
- 每个检查点提交消息必须描述阶段和捕获的确切证据
- 仅统计当前活跃分支上为当前任务创建的提交
- 不要将其他分支、早期无关工作或远端分支历史的提交视为有效检查点证据
- 将检查点视为已满足前，验证该提交可从当前活跃分支的 `HEAD` 到达，且属于当前任务序列
- 推荐的精简工作流：
  - 一个提交用于添加失败测试并验证 RED
  - 一个提交用于应用最小修复并验证 GREEN
  - 一个可选提交用于完成重构
- 若测试提交明确对应 RED、修复提交明确对应 GREEN，则不需要单独的证据提交

## TDD 工作流步骤

### 第 1 步：编写用户旅程
```
作为 [角色]，我希望 [操作]，以便 [收益]

示例：
作为用户，我希望能够语义搜索市场，
以便即使没有精确关键词也能找到相关市场。
```

### 第 2 步：生成测试用例
针对每个用户旅程，创建全面的测试用例：

```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => {
    // 测试实现
  })

  it('returns empty array for no matches', async () => {})
  it('handles special characters in query', async () => {})
  it('returns results sorted by relevance', async () => {})
})
```

### 第 3 步：RED — 运行测试（必须失败）
```bash
npm test -- --testPathPattern="semantic-search"
# 预期：FAIL — 功能尚不存在
```

**关键**：在继续前确认测试失败。若测试通过，说明测试有误。

### 第 4 步：GREEN — 编写最小实现
仅编写足以让测试通过的代码：

```typescript
// 最小实现 — 仅让测试通过
export async function semanticSearch(query: string) {
  if (!query) return []
  return await db.markets.findBySemanticSimilarity(query)
}
```

再次运行测试：
```bash
npm test -- --testPathPattern="semantic-search"
# 预期：PASS
```

### 第 5 步：REFACTOR — 改进代码
测试通过后，在保持测试绿色的同时改进代码：

```typescript
// 重构后 — 更好的错误处理和类型
export async function semanticSearch(
  query: string,
  options: SearchOptions = {}
): Promise<Market[]> {
  if (!query.trim()) return []

  const { limit = 10, threshold = 0.7 } = options
  return await db.markets.findBySemanticSimilarity(query, { limit, threshold })
}
```

### 第 6 步：验证覆盖率
```bash
npm test -- --coverage --testPathPattern="semantic-search"
# 目标：80%+ 覆盖率
```

## 测试模式

### 单元测试模式

```typescript
// AAA 模式：Arrange-Act-Assert
describe('calculateFee', () => {
  it('returns 2% fee for standard trade', () => {
    // Arrange
    const amount = 100
    const tradeType = 'standard'

    // Act
    const fee = calculateFee(amount, tradeType)

    // Assert
    expect(fee).toBe(2)
  })

  it('returns 0 for zero amount', () => {
    expect(calculateFee(0, 'standard')).toBe(0)
  })

  it('throws for negative amount', () => {
    expect(() => calculateFee(-1, 'standard')).toThrow('Amount must be positive')
  })
})
```

### 集成测试模式

```typescript
describe('POST /api/markets', () => {
  beforeEach(async () => {
    await db.markets.deleteMany({})
  })

  it('creates market with valid data', async () => {
    const response = await request(app)
      .post('/api/markets')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        title: 'Test Market',
        description: 'Test description',
        endDate: '2024-12-31'
      })

    expect(response.status).toBe(201)
    expect(response.body.data.title).toBe('Test Market')
  })

  it('returns 400 for missing required fields', async () => {
    const response = await request(app)
      .post('/api/markets')
      .set('Authorization', `Bearer ${testToken}`)
      .send({})

    expect(response.status).toBe(400)
    expect(response.body.errors).toBeDefined()
  })
})
```

### Mock 模式

```typescript
// Mock 外部服务
jest.mock('@/lib/openai', () => ({
  createEmbedding: jest.fn().mockResolvedValue([0.1, 0.2, 0.3])
}))

// Mock 数据库
jest.mock('@/lib/db', () => ({
  markets: {
    findBySemanticSimilarity: jest.fn().mockResolvedValue([
      { id: '1', title: 'Test Market', similarity: 0.9 }
    ])
  }
}))

// 测试中验证 mock 调用
it('calls embedding service with query', async () => {
  await semanticSearch('test query')
  expect(createEmbedding).toHaveBeenCalledWith('test query')
})
```

## 覆盖率要求

```typescript
// jest.config.ts
export default {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
}
```

## Pre-commit Hooks

```bash
# 每次提交前运行
npm test && npm run lint
```

### CI/CD 集成
```yaml
# GitHub Actions
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## 最佳实践

1. **先写测试** — 始终 TDD
2. **每个测试一个断言** — 专注于单一行为
3. **描述性测试名称** — 解释测试内容
4. **Arrange-Act-Assert** — 清晰的测试结构
5. **Mock 外部依赖** — 隔离单元测试
6. **测试边界情况** — Null、undefined、空值、大值
7. **测试错误路径** — 不只是 happy path
8. **保持测试快速** — 单元测试每个 < 50ms
9. **测试后清理** — 无副作用
10. **审查覆盖率报告** — 识别缺口

## 成功指标

- 达到 80%+ 代码覆盖率
- 所有测试通过（绿色）
- 无跳过或禁用的测试
- 测试执行快速（单元测试 < 30s）
- E2E 测试覆盖关键用户流程
- 测试在生产前捕获 bug

---

**记住**：测试不是可选的。它们是安全网，使自信重构、快速开发和生产可靠性成为可能。
