---
name: backend-patterns
description: 后端架构模式、API 设计、数据库优化以及 Node.js、Express 和 Next.js API 路由的服务器端最佳实践。
origin: ECC
---

# 后端开发模式

面向可扩展服务端应用的后端架构模式与最佳实践。

## 何时激活

- 设计 REST 或 GraphQL API 端点
- 实现仓库层、服务层或控制器层
- 优化数据库查询（N+1 问题、索引、连接池）
- 添加缓存（Redis、内存缓存、HTTP 缓存头）
- 设置后台任务或异步处理
- 构建 API 的错误处理和验证结构
- 构建中间件（身份验证、日志记录、速率限制）

## API 设计模式

### RESTful API 结构

```typescript
// 通过：基于资源的 URL
GET    /api/markets                 # 列出资源
GET    /api/markets/:id             # 获取单个资源
POST   /api/markets                 # 创建资源
PUT    /api/markets/:id             # 替换资源
PATCH  /api/markets/:id             # 更新资源
DELETE /api/markets/:id             # 删除资源

// 通过：用于过滤、排序、分页的查询参数
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### 仓库模式

```typescript
// 抽象数据访问逻辑
interface MarketRepository {
findAll(filters?: MarketFilters): Promise<Market[]>
findById(id: string): Promise<Market | null>
create(data: CreateMarketDto): Promise<Market>
update(id: string, data: UpdateMarketDto): Promise<Market>
delete(id: string): Promise<void>
}

class SupabaseMarketRepository 实现了 MarketRepository {
async findAll(filters?: MarketFilters): Promise<Market[]> {
let query = supabase.from('markets').select('*')

if (filters?.status) {
query = query.eq('status', filters.status)
}

如果 (filters?.limit) {
query = query.limit(filters.limit)
}

const { data, error } = await query

if (error) throw new Error(error.message)
return data
}

// 其他方法...
}
```

### 服务层模式

```typescript
// 业务逻辑与数据访问分离
class MarketService {
constructor(private marketRepo: MarketRepository) {}

async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
// 业务逻辑
const embedding = await generateEmbedding(query)
const results = await this.vectorSearch(embedding, limit)

// 获取完整数据
const markets = await this.marketRepo.findByIds(results.map(r => r.id))

// 按相似度排序
return markets.sort((a, b) => {
const scoreA = results.find(r => r.id === a.id)?.score || 0
const scoreB = results.find(r => r.id === b.id)?.score || 0
return scoreA - scoreB
})
}

private async vectorSearch(embedding: number[], limit: number) {
// 向量搜索实现
}
}
```

### 中间件模式

```typescript
// 请求/响应处理管道
导出函数 withAuth(handler: NextApiHandler): NextApiHandler {
返回异步函数 (req, res) => {
从请求头中提取令牌：const token = req.headers.authorization?.replace('Bearer ', '')

如果令牌不存在：if (!token) {
返回 res.status(401).json({ error: '未授权' })
}

try {
const user = await verifyToken(token)
req.user = user
return handler(req, res)
} catch (error) {
return res.status(401).json({ error: '无效的令牌' })
}
}
}

// 使用示例
export default withAuth(async (req, res) => {
// 处理程序可以访问 req.user
})
好的，这是您要的译文：

```html
<p>你好，世界！</p>
```

## 数据库模式

### 查询优化

```typescript
// 通过：良好：仅选择需要的列
const { data } = await supabase
.from('markets')
.select('id, name, status, volume')
.eq('status', 'active')
.order('volume', { ascending: false })
.limit(10)

// 失败：错误：选择所有内容
const { data } = await supabase
.from('markets')
.select('*')
```

### N+1 查询预防

```typescript
// 失败：错误：N+1 查询问题
const markets = await getMarkets()
for (const market of markets) {
market.creator = await getUser(market.creator_id)  // N 次查询
}

// 通过：良好：批量获取
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 1 次查询
const creatorMap = new Map(creators.map(c => [c.id, c]))

markets.forEach(market => {
market.creator = creatorMap.get(market.creator_id)
})
```

### 事务模式

```typescript
async function createMarketWithPosition(
marketData: CreateMarketDto,
positionData: CreatePositionDto
) {
// 使用 Supabase 事务
const { data, error } = await supabase.rpc('create_market_with_position', {
market_data: marketData,
position_data: positionData
})

if (error) throw new Error('交易失败')
return data
}

// Supabase 中的 SQL 函数
CREATE OR REPLACE FUNCTION create_market_with_position(
market_data jsonb,
position_data jsonb
)
返回 jsonb
语言 plpgsql
AS $$
BEGIN
-- 自动开始事务
INSERT INTO markets VALUES (market_data);
INSERT INTO positions VALUES (position_data);
RETURN jsonb_build_object('success', true);
EXCEPTION
WHEN OTHERS THEN
-- 回滚自动发生
RETURN jsonb_build_object('success', false, 'error', SQLERRM);
END;
$$;
```

## 缓存策略

### Redis 缓存层

```typescript
class CachedMarketRepository 实现了 MarketRepository {
构造函数(
私有 baseRepo: MarketRepository,
私有 redis: RedisClient
) {}

async findById(id: string): Promise<Market | null> {
// 优先检查缓存
const cached = await this.redis.get(`market:${id}`)

如果（已缓存）{
返回 JSON.parse(缓存)
}

// 缓存未命中 - 从数据库获取
const market = await this.baseRepo.findById(id)

if (market) {
// 缓存 5 分钟
await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
}

return market
}

async invalidateCache(id: string): Promise<void> {
await this.redis.del(`market:${id}`)
}
}
```

### 缓存旁路模式

```typescript
async function getMarketWithCache(id: string): Promise<Market> {
const cacheKey = `market:${id}`

// 尝试缓存
const cached = await redis.get(cacheKey)
if (cached) return JSON.parse(cached)

// 缓存未命中 - 从数据库获取
const market = await db.markets.findUnique({ where: { id } })

if (!market) throw new Error('未找到市场')

// 更新缓存
await redis.setex(cacheKey, 300, JSON.stringify(market))

返回市场
}
```

## 错误处理模式

### 集中式错误处理器

```typescript
class ApiError extends Error {
constructor(
public statusCode: number,
public message: string,
public isOperational = true
) {
super(message)
Object.setPrototypeOf(this, ApiError.prototype)
}
}

export function errorHandler(error: unknown, req: Request): Response {
if (error instanceof ApiError) {
return NextResponse.json({
success: false,
错误：错误信息
}, { 状态：错误状态码 })
}

如果（错误 instanceof z.ZodError）{
返回 NextResponse.json({
success: false,
error: '验证失败',
details: error.errors
}, { status: 400 })
}

// 记录意外错误
console.error('意外错误：', error)

返回 NextResponse.json({
success: false,
error: '服务器内部错误'
}, { status: 500 })
}

// 用法
export async function GET(request: Request) {
try {
const data = await fetchData()
return NextResponse.json({ success: true, data })
} catch (error) {
return errorHandler(error, request)
}
}
```

### 使用指数退避重试

```typescript
async function fetchWithRetry<T>(
fn: () => Promise<T>,
maxRetries = 3
): Promise<T> {
let lastError: Error

for (let i = 0; i < maxRetries; i++) {
try {
返回 await fn()
} catch (error) {
lastError = error as Error

if (i < maxRetries - 1) {
// 指数退避：1 秒、2 秒、4 秒
const delay = Math.pow(2, i) * 1000
await new Promise(resolve => setTimeout(resolve, delay))
}
}
}

抛出最后的错误！
}

// 用法
const data = await fetchWithRetry(() => fetchFromAPI())
```

## 身份验证与授权

### JWT 令牌验证

```typescript
import jwt from 'jsonwebtoken'

interface JWTPayload {
userId: string
email: string
role: 'admin' | 'user'
}

export function verifyToken(token: string): JWTPayload {
try {
const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
return payload
} catch (error) {
throw new ApiError(401, '无效令牌')
}
}

export async function requireAuth(request: Request) {
const token = request.headers.get('authorization')?.replace('Bearer ', '')

if (!token) {
throw new ApiError(401, '缺少授权令牌')
}

return verifyToken(token)
}

// 在 API 路由中使用
export async function GET(request: Request) {
const user = await requireAuth(request)

const data = await getDataForUser(user.userId)

return NextResponse.json({ success: true, data })
}
```

### 基于角色的访问控制

```typescript
type Permission = 'read' | 'write' | 'delete' | 'admin'

interface User {
id: string
role: 'admin' | 'moderator' | 'user'
}

const rolePermissions: Record<User['role'], Permission[]> = {
admin: ['read', 'write', 'delete', 'admin'],
moderator: ['read', 'write', 'delete'],
user: ['read', 'write']
}

export function hasPermission(user: User, permission: Permission): boolean {
return rolePermissions[user.role].includes(permission)
}

export function requirePermission(permission: Permission) {
return (handler: (request: Request, user: User) => Promise<Response>) => {
return async (request: Request) => {
const user = await requireAuth(request)

if (!hasPermission(user, permission)) {
throw new ApiError(403, '权限不足')
}

返回处理程序(请求, 用户)
}
}
}

// 用法 - 高阶函数包装处理器
export const DELETE = requirePermission('delete')(
async (request: Request, user: User) => {
// 处理器接收已验证权限的认证用户
返回新的响应（'已删除'，{ 状态：200 }）
}
)
```

## 速率限制

速率限制必须使用共享存储（如 Redis）、网关或
平台的原生限流器。请勿在生产环境的 API 中使用基于进程的内存计数器：
它们会在部署时重置、在副本间分散，并且默认以开放模式失效。
无服务器或多实例环境。

保持后端层负责选择集成点和错误处理。
形态；使用 `api-design` 处理 HTTP 契约，使用 `security-review` 进行滥用案例审查。
案例审查。

## 后台任务与队列

### 简单队列模式

```typescript
class JobQueue<T> {
private queue: T[] = []
private processing = false

async add(job: T): Promise<void> {
this.queue.push(job)

如果 (!this.processing) {
this.process()
}
}

private async process(): Promise<void> {
this.processing = true

while (this.queue.length > 0) {
const job = this.queue.shift()!

尝试 {
await this.execute(job)
} 捕获 (错误) {
console.error('任务失败:', error)
}
}

this.processing = false
}

private async execute(job: T): Promise<void> {
// 任务执行逻辑
}
}

// 用于索引市场的用法
interface IndexJob {
marketId: string
}

const indexQueue = new JobQueue<IndexJob>()

export async function POST(request: Request) {
const { marketId } = await request.json()

// 加入队列而非阻塞
await indexQueue.add({ marketId })

return NextResponse.json({ success: true, message: '任务已加入队列' })
}
```

## 日志记录与监控

### 结构化日志

```typescript
interface LogContext {
userId?: string
requestId?: string
method?: string
path?: string
[key: string]: unknown
}

class Logger {
log(level: 'info' | 'warn' | 'error', message: string, context?: LogContext) {
const entry = {
timestamp: new Date().toISOString(),
level,
message,
...上下文
}

console.log(JSON.stringify(entry))
}

info(message: string, context?: LogContext) {
this.log('info', message, context)
}

warn(message: string, context?: LogContext) {
this.log('warn', message, context)
}

error(message: string, error: Error, context?: LogContext) {
this.log('error', message, {
...context,
error: error.message,
stack: error.stack
})
}
}

const logger = new Logger()

// 使用
export async function GET(request: Request) {
const requestId = crypto.randomUUID()

logger.info('正在获取市场数据', {
requestId,
方法：'GET'，
路径：'/api/markets'
})

尝试 {
const markets = await fetchMarkets()
return NextResponse.json({ success: true, data: markets })
} catch (error) {
logger.error('获取市场数据失败', error as Error, { requestId })
返回 NextResponse.json({ error: '内部错误' }, { status: 500 })
}
}
```

**记住**：后端模式能够构建可扩展、可维护的服务端应用。请选择适合你项目复杂度的模式。