---
name: frontend-patterns
description: React、Next.js、Vue 2 / Vue 3、状态管理、性能优化和 UI 最佳实践的前端开发模式。
origin: ECC
---

# 前端开发模式

适用于 React、Next.js、Vue 2 / Vue 3、高性能 UI 现代前端模式。

## 何时激活
- 构建 React 组件（组合、属性、渲染）
- 构建 Vue 2 / Vue 3 组件（Options API、Composition API、插槽、响应式）
- 管理状态（useState、useReducer、Zustand、Context、Vuex、Pinia）
- 实现数据获取（SWR、React Query、服务器组件、Vue composables）
- 优化性能（记忆化、虚拟化、代码拆分）
- 处理表单（验证、受控输入、Zod 模式）
- 处理客户端路由和导航
- 构建可访问、响应式 UI 模式

## 使用原则
- 组合优于继承
- 状态尽量靠近使用点，跨层共享再提升
- 服务端状态与客户端状态分离
- 先保证可读性，再做性能优化
- 交互组件默认包含键盘支持、焦点管理、语义化标签
- 选择适合项目复杂度模式，不为简单页面引入重状态管理

## 组件模式
### 组合优于继承（React）
```typescript
interface CardProps {
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}
export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}
export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}
export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}
<Card>
  <CardHeader>标题</CardHeader>
  <CardBody>内容</CardBody>
</Card>
```

### 插槽组合（Vue 2 / Vue 3）
```vue
<template>
  <div :class="`card card-${variant}`">
    <header class="card-header"><slot name="header" /></header>
    <div class="card-body"><slot /></div>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: { variant: { type: String, default: 'default' } }
})
</script>
```

```vue
<script setup lang="ts">
interface CardProps { variant?: 'default' | 'outlined' }
const props = withDefaults(defineProps<CardProps>(), { variant: 'default' })
</script>
<template>
  <div :class="`card card-${props.variant}`">
    <header class="card-header"><slot name="header" /></header>
    <div class="card-body"><slot /></div>
  </div>
</template>
```

### 复合组件（React）
```typescript
interface TabsContextValue {
  activeTab: string
  setActiveTab: (tab: string) => void
}
const TabsContext = createContext<TabsContextValue | undefined>(undefined)
export function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab)
  return <TabsContext.Provider value={{ activeTab, setActiveTab }}>{children}</TabsContext.Provider>
}
export function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>
}
export function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const context = useContext(TabsContext)
  if (!context) throw new Error('Tab must be used within Tabs')
  return <button className={context.activeTab === id ? 'active' : ''} onClick={() => context.setActiveTab(id)}>{children}</button>
}
<Tabs defaultTab="overview"><TabList><Tab id="overview">概览</Tab><Tab id="details">详情</Tab></TabList></Tabs>
```

### 复合组件骨架（Vue 2 / Vue 3）
```vue
<template><div class="tabs-root"><slot :active-tab="activeTab" :set-active-tab="setActiveTab" /></div></template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: { defaultTab: { type: String, required: true } },
  data() { return { activeTab: this.defaultTab } },
  methods: { setActiveTab(tab: string) { this.activeTab = tab } }
})
</script>
```

```vue
<script setup lang="ts">
import { computed, provide, ref } from 'vue'
const props = defineProps<{ defaultTab: string }>()
const activeTab = ref(props.defaultTab)
provide('tabsContext', { activeTab: computed(() => activeTab.value), setActiveTab: (tab: string) => { activeTab.value = tab } })
</script>
<template><div class="tabs-root"><slot /></div></template>
```

### Render Props / 作用域插槽
```typescript
interface DataLoaderProps<T> {
  url: string
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode
}
export function DataLoader<T>({ url, children }: DataLoaderProps<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)
  useEffect(() => {
    fetch(url).then(res => res.json()).then(setData).catch(setError).finally(() => setLoading(false))
  }, [url])
  return <>{children(data, loading, error)}</>
}
<DataLoader<Market[]> url="/api/markets">{(markets, loading, error) => {
  if (loading) return <Spinner />
  if (error) return <Error error={error} />
  return <MarketList markets={markets!} />
}}</DataLoader>
```

```vue
<script setup lang="ts" generic="T">
import { onMounted, ref, watch } from 'vue'
const props = defineProps<{ url: string }>()
const data = ref<T | null>(null)
const loading = ref(true)
const error = ref<Error | null>(null)
const load = async () => {
  loading.value = true
  error.value = null
  try {
    const response = await fetch(props.url)
    data.value = await response.json()
  } catch (err) {
    error.value = err as Error
  } finally {
    loading.value = false
  }
}
watch(() => props.url, load)
onMounted(load)
</script>
<template><slot :data="data" :loading="loading" :error="error" /></template>
```

## 自定义 Hooks / Composables 模式
### 状态管理 Hook / Composable
```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue(v => !v), [])
  return [value, toggle]
}
const [isOpen, toggleOpen] = useToggle()
```

```typescript
import { ref } from 'vue'
export function useToggle(initialValue = false) {
  const value = ref(initialValue)
  const toggle = () => { value.value = !value.value }
  return { value, toggle }
}
const { value: isOpen, toggle } = useToggle()
```

### 异步数据获取
```typescript
interface UseQueryOptions<T> {
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
  enabled?: boolean
}
export function useQuery<T>(key: string, fetcher: () => Promise<T>, options?: UseQueryOptions<T>) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(false)
  const refetch = useCallback(async () => {
    setLoading(true)
    setError(null)
    try {
      const result = await fetcher()
      setData(result)
      options?.onSuccess?.(result)
    } catch (err) {
      const error = err as Error
      setError(error)
      options?.onError?.(error)
    } finally {
      setLoading(false)
    }
  }, [fetcher, options])
  useEffect(() => {
    if (options?.enabled !== false) refetch()
  }, [key, refetch, options?.enabled])
  return { data, error, loading, refetch }
}
const { data: markets, loading, error, refetch } = useQuery('markets', () => fetch('/api/markets').then(r => r.json()), {
  onSuccess: data => console.log('获取到', data.length, '个市场'),
  onError: err => console.error('失败:', err)
})
```

```typescript
export default {
  data() {
    return { markets: [], loading: false, error: null as Error | null }
  },
  watch: {
    marketId: {
      immediate: true,
      async handler() {
        this.loading = true
        this.error = null
        try {
          const response = await fetch(`/api/markets/${this.marketId}`)
          this.markets = await response.json()
        } catch (err) {
          this.error = err as Error
        } finally {
          this.loading = false
        }
      }
    }
  }
}
```

```typescript
import { ref, watch } from 'vue'
interface UseQueryOptions<T> {
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
  enabled?: boolean
}
export function useQuery<T>(key: () => string, fetcher: () => Promise<T>, options?: UseQueryOptions<T>) {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)
  const refetch = async () => {
    loading.value = true
    error.value = null
    try {
      const result = await fetcher()
      data.value = result
      options?.onSuccess?.(result)
    } catch (err) {
      const resolvedError = err as Error
      error.value = resolvedError
      options?.onError?.(resolvedError)
    } finally {
      loading.value = false
    }
  }
  watch(key, () => {
    if (options?.enabled !== false) void refetch()
  }, { immediate: true })
  return { data, error, loading, refetch }
}
```

### 防抖
```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)
  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])
  return debouncedValue
}
const [searchQuery, setSearchQuery] = useState('')
const debouncedQuery = useDebounce(searchQuery, 500)
useEffect(() => {
  if (debouncedQuery) performSearch(debouncedQuery)
}, [debouncedQuery])
```

```typescript
export default {
  data() {
    return { searchQuery: '', debouncedQuery: '' }
  },
  created() {
    this.applyDebounce = this.createDebounce(500)
  },
  watch: {
    searchQuery(value: string) {
      this.applyDebounce(value)
    }
  },
  methods: {
    createDebounce(delay: number) {
      let handler = 0
      return (value: string) => {
        window.clearTimeout(handler)
        handler = window.setTimeout(() => { this.debouncedQuery = value }, delay)
      }
    }
  }
}
```

```typescript
import { ref, watch } from 'vue'
export function useDebounce<T>(value: () => T, delay: number) {
  const debouncedValue = ref(value())
  watch(value, nextValue => {
    const handler = window.setTimeout(() => { debouncedValue.value = nextValue }, delay)
    return () => window.clearTimeout(handler)
  })
  return debouncedValue
}
```

## 状态管理模式
### Context + Reducer
```typescript
interface State {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
}
type Action =
  | { type: 'SET_MARKETS'; payload: Market[] }
  | { type: 'SELECT_MARKET'; payload: Market }
  | { type: 'SET_LOADING'; payload: boolean }
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_MARKETS':
      return { ...state, markets: action.payload }
    case 'SELECT_MARKET':
      return { ...state, selectedMarket: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
    default:
      return state
  }
}
const MarketContext = createContext<{ state: State; dispatch: Dispatch<Action> } | undefined>(undefined)
export function MarketProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, { markets: [], selectedMarket: null, loading: false })
  return <MarketContext.Provider value={{ state, dispatch }}>{children}</MarketContext.Provider>
}
export function useMarkets() {
  const context = useContext(MarketContext)
  if (!context) throw new Error('useMarkets must be used within MarketProvider')
  return context
}
```

### Vuex / Pinia
```typescript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
interface State {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
}
export default new Vuex.Store<State>({
  state: { markets: [], selectedMarket: null, loading: false },
  mutations: {
    SET_MARKETS(state, markets: Market[]) { state.markets = markets },
    SELECT_MARKET(state, market: Market) { state.selectedMarket = market },
    SET_LOADING(state, loading: boolean) { state.loading = loading }
  },
  actions: {
    async loadMarkets({ commit }) {
      commit('SET_LOADING', true)
      try {
        const response = await fetch('/api/markets')
        commit('SET_MARKETS', await response.json())
      } finally {
        commit('SET_LOADING', false)
      }
    }
  }
})
```

```typescript
import { defineStore } from 'pinia'
export const useMarketStore = defineStore('markets', {
  state: () => ({ markets: [] as Market[], selectedMarket: null as Market | null, loading: false }),
  actions: {
    async loadMarkets() {
      this.loading = true
      try {
        const response = await fetch('/api/markets')
        this.markets = await response.json()
      } finally {
        this.loading = false
      }
    }
  }
})
```

## 性能优化
### 记忆化
```typescript
const sortedMarkets = useMemo(() => markets.sort((a, b) => b.volume - a.volume), [markets])
const handleSearch = useCallback((query: string) => { setSearchQuery(query) }, [])
export const MarketCard = React.memo<MarketCardProps>(({ market }) => {
  return <div className="market-card"><h3>{market.name}</h3><p>{market.description}</p></div>
})
```

### Vue 计算属性与稳定引用
```vue
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: { markets: { type: Array, required: true } },
  computed: { sortedMarkets(): Market[] { return [...this.markets].sort((a, b) => b.volume - a.volume) } }
})
</script>
```

```typescript
import { computed } from 'vue'
const sortedMarkets = computed(() => [...markets.value].sort((a, b) => b.volume - a.volume))
const marketSummary = computed(() => ({ count: markets.value.length, totalVolume: markets.value.reduce((sum, market) => sum + market.volume, 0) }))
```

### 代码拆分与懒加载
```typescript
import { lazy, Suspense } from 'react'
const HeavyChart = lazy(() => import('./HeavyChart'))
const ThreeJsBackground = lazy(() => import('./ThreeJsBackground'))
export function Dashboard() {
  return <div><Suspense fallback={<ChartSkeleton />}><HeavyChart data={data} /></Suspense><Suspense fallback={null}><ThreeJsBackground /></Suspense></div>
}
```

```typescript
import Vue from 'vue'
export default Vue.extend({
  components: {
    HeavyChart: () => import('./HeavyChart.vue'),
    ThreeJsBackground: () => import('./ThreeJsBackground.vue')
  }
})
```

### 长列表虚拟化
```typescript
import { useVirtualizer } from '@tanstack/react-virtual'
export function VirtualMarketList({ markets }: { markets: Market[] }) {
  const parentRef = useRef<HTMLDivElement>(null)
  const virtualizer = useVirtualizer({ count: markets.length, getScrollElement: () => parentRef.current, estimateSize: () => 100, overscan: 5 })
  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div key={virtualRow.index} style={{ position: 'absolute', top: 0, left: 0, width: '100%', height: `${virtualRow.size}px`, transform: `translateY(${virtualRow.start}px)` }}>
            <MarketCard market={markets[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

```vue
<template>
  <RecycleScroller class="market-list" :items="markets" :item-size="100" key-field="id">
    <template #default="{ item }"><MarketCard :market="item" /></template>
  </RecycleScroller>
</template>
<script lang="ts">
import Vue from 'vue'
import { RecycleScroller } from 'vue-virtual-scroller'
export default Vue.extend({
  components: { RecycleScroller },
  props: { markets: { type: Array, required: true } }
})
</script>
```

## 表单处理模式
### 带验证表单
```typescript
interface FormData {
  name: string
  description: string
  endDate: string
}
interface FormErrors {
  name?: string
  description?: string
  endDate?: string
}
export function CreateMarketForm() {
  const [formData, setFormData] = useState<FormData>({ name: '', description: '', endDate: '' })
  const [errors, setErrors] = useState<FormErrors>({})
  const validate = (): boolean => {
    const newErrors: FormErrors = {}
    if (!formData.name.trim()) newErrors.name = '名称是必填的'
    else if (formData.name.length > 200) newErrors.name = '名称必须少于 200 个字符'
    if (!formData.description.trim()) newErrors.description = '描述是必填的'
    if (!formData.endDate) newErrors.endDate = '结束日期是必填的'
    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    if (!validate()) return
    try {
      await createMarket(formData)
    } catch (error) {
      // 错误处理
    }
  }
  return (
    <form onSubmit={handleSubmit}>
      <input value={formData.name} onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))} placeholder="市场名称" />
      {errors.name && <span className="error">{errors.name}</span>}
      <button type="submit">创建市场</button>
    </form>
  )
}
```

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model.trim="form.name" placeholder="市场名称">
    <span v-if="errors.name" class="error">{{ errors.name }}</span>
    <textarea v-model.trim="form.description" />
    <span v-if="errors.description" class="error">{{ errors.description }}</span>
    <button type="submit">创建市场</button>
  </form>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  data() {
    return { form: { name: '', description: '' }, errors: {} as Record<string, string> }
  },
  methods: {
    validate() {
      const errors: Record<string, string> = {}
      if (!this.form.name) errors.name = '名称是必填的'
      if (!this.form.description) errors.description = '描述是必填的'
      this.errors = errors
      return Object.keys(errors).length === 0
    },
    async handleSubmit() {
      if (!this.validate()) return
      await createMarket(this.form)
    }
  }
})
</script>
```

## 错误边界模式
```typescript
interface ErrorBoundaryState {
  hasError: boolean
  error: Error | null
}
export class ErrorBoundary extends React.Component<{ children: React.ReactNode }, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false, error: null }
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('错误边界捕获:', error, errorInfo)
  }
  render() {
    if (this.state.hasError) {
      return <div className="error-fallback"><h2>出错了</h2><p>{this.state.error?.message}</p><button onClick={() => this.setState({ hasError: false })}>重试</button></div>
    }
    return this.props.children
  }
}
<ErrorBoundary><App /></ErrorBoundary>
```

## 动画模式
```typescript
import { motion, AnimatePresence } from 'framer-motion'
export function AnimatedMarketList({ markets }: { markets: Market[] }) {
  return <AnimatePresence>{markets.map(market => <motion.div key={market.id} initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -20 }} transition={{ duration: 0.3 }}><MarketCard market={market} /></motion.div>)}</AnimatePresence>
}
export function Modal({ isOpen, onClose, children }: ModalProps) {
  return (
    <AnimatePresence>
      {isOpen && <><motion.div className="modal-overlay" initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} onClick={onClose} /><motion.div className="modal-content" initial={{ opacity: 0, scale: 0.9, y: 20 }} animate={{ opacity: 1, scale: 1, y: 0 }} exit={{ opacity: 0, scale: 0.9, y: 20 }}>{children}</motion.div></>}
    </AnimatePresence>
  )
}
```

## 可访问性模式
### 键盘导航
```typescript
export function Dropdown({ options, onSelect }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false)
  const [activeIndex, setActiveIndex] = useState(0)
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault()
        setActiveIndex(i => Math.min(i + 1, options.length - 1))
        break
      case 'ArrowUp':
        e.preventDefault()
        setActiveIndex(i => Math.max(i - 1, 0))
        break
      case 'Enter':
        e.preventDefault()
        onSelect(options[activeIndex])
        setIsOpen(false)
        break
      case 'Escape':
        setIsOpen(false)
        break
    }
  }
  return <div role="combobox" aria-expanded={isOpen} aria-haspopup="listbox" onKeyDown={handleKeyDown}>{/* 下拉菜单实现 */}</div>
}
```

```vue
<template>
  <div role="combobox" :aria-expanded="isOpen" aria-haspopup="listbox" @keydown.down.prevent="move(1)" @keydown.up.prevent="move(-1)" @keydown.enter.prevent="selectActive" @keydown.esc="isOpen = false">
    <slot />
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: { options: { type: Array, required: true } },
  data() { return { isOpen: false, activeIndex: 0 } },
  methods: {
    move(step: number) {
      const nextIndex = this.activeIndex + step
      this.activeIndex = Math.max(0, Math.min(nextIndex, this.options.length - 1))
    },
    selectActive() {
      this.$emit('select', this.options[this.activeIndex])
      this.isOpen = false
    }
  }
})
</script>
```

### 焦点管理
```typescript
export function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null)
  const previousFocusRef = useRef<HTMLElement | null>(null)
  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement as HTMLElement
      modalRef.current?.focus()
    } else {
      previousFocusRef.current?.focus()
    }
  }, [isOpen])
  return isOpen ? <div ref={modalRef} role="dialog" aria-modal="true" tabIndex={-1} onKeyDown={e => e.key === 'Escape' && onClose()}>{children}</div> : null
}
```

```vue
<template>
  <div v-if="isOpen" ref="modal" role="dialog" aria-modal="true" tabindex="-1" @keydown.esc="$emit('close')"><slot /></div>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: { isOpen: { type: Boolean, required: true } },
  data() { return { previousFocus: null as HTMLElement | null } },
  watch: {
    isOpen(value: boolean) {
      if (value) {
        this.previousFocus = document.activeElement as HTMLElement
        this.$nextTick(() => { ;(this.$refs.modal as HTMLElement).focus() })
      } else {
        this.previousFocus?.focus()
      }
    }
  }
})
</script>
```

**记住**：现代前端模式能构建可维护、高性能 UI。选适合项目复杂度模式。
