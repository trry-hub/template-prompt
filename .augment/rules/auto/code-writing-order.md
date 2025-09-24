---
type: "agent_requested"
description: "代码编写顺序"
---

# Vue 3 + TypeScript 代码书写顺序规范

## 🚨 严格执行顺序

### 1. 导入声明 (Import Statements)

```vue
<script setup lang="ts">
import type { EvaluateDesc, EvaluateInfo } from './evaluate-settings'
import type { FieldConfigMap } from '@/components/AttributeFilter/types'

import { ElMessage, ElMessageBox } from 'element-plus'

import { someApi } from '@/api/modules/example'
import { useAuth, usePagination } from '@/utils/composables'

import ChildComponent from './components/ChildComponent.vue'
import EvaluateSettings from './evaluate-settings.vue'
```

### 2. 类型定义 (Type Definitions)

```typescript
interface FormData {
  name: string
  age: number
}

type Status = 'pending' | 'success' | 'error'

enum UserRole {
  ADMIN = 'admin',
  USER = 'user'
}
```

### 3. 组件配置 (Component Configuration)

```typescript
defineOptions({
  name: 'ComponentName',
  inheritAttrs: false
})

// Emits 定义 - 必须在 Props 之前
const emit = defineEmits<{
  delete: []
  change: [value: string]
  submit: [data: FormData]
}>()

const {
  id,
  config = {},
  disabled = false
} = defineProps<{
  id: string
  config?: object
  disabled?: boolean
}>()
```

### 4. 组合式函数 (Composables/Hooks)

```typescript
const router = useRouter()
const route = useRoute()

const { user, hasPermission } = useAuth()
const { pagination, handlePagination } = usePagination()

const { copy } = useClipboard()
```

### 5. 常量和配置 (Constants & Configuration)

```typescript
const MAX_ITEMS = 2
const DEFAULT_PAGE_SIZE = 10

const fieldConfig = {
  titleIds: {
    label: '职称等级：',
    placeholder: '请选择',
    elementClass: 'customize-v3-cascader',
  }
}

const defaultFormData = {
  name: '',
  age: 0
}
```

### 6. 响应式数据 (Reactive Data)

```typescript
const loading = ref(false)
const visible = ref(false)

const formData = ref<FormData>({
  name: '',
  age: 0
})

const tableData = reactive({
  list: [],
  total: 0
})

const baseEvaluateInfo = ref<EvaluateInfo>({
  title: '示例标题',
  date: '2025-01-01',
  tags: [],
  participantInfo: {
    totalCount: 0,
    targetRatio: 0,
    completionRatio: 0,
  },
  timeRange: ['', '']
})

const comparisonItems = ref<Array<EvaluateInfo | null>>([])
```

### 7. 计算属性 (Computed Properties)

```typescript
const isFormValid = computed(() => {
  return formData.value.name && formData.value.age > 0
})

const canAddMore = computed(() => comparisonItems.value.length < MAX_ITEMS)

const formattedDate = computed(() => {
  return dayjs(formData.value.date).format('YYYY-MM-DD')
})
```

### 8. 方法定义 (Methods)

```typescript
async function loadData() {
  loading.value = true

  const { res, error } = await useApi(someApi, { id })

  if (res) {
    tableData.list = res.data
    tableData.total = res.total
    ElMessage.success('数据加载成功')
  }

  if (error) {
    ElMessage.error(error.message || '加载失败')
  }

  loading.value = false
}

function handleConfirm(data: any) {
  dialogVisible.value = false
  loadTargetData(data)
}

async function loadTargetData(data: any) {
  targetLoading.value = true

  const { res, error } = await useApi(targetApi, data)

  if (res) {
    targetData.value = res.data
    ElMessage.success('操作成功')
  }

  if (error) {
    ElMessage.error(error.message || '操作失败')
  }

  targetLoading.value = false
}

function handleSubmit() {
  if (!isFormValid.value) {
    ElMessage.warning('请填写完整信息')
    return
  }
  emit('submit', formData.value)
}

function handleDelete() {
  emit('delete')
}

function addComparisonGroup() {
  if (comparisonItems.value.length < MAX_ITEMS) {
    comparisonItems.value.push(null)
  }
}

function removeComparisonGroup(index: number) {
  comparisonItems.value.splice(index, 1)
}
```

### 9. 监听器 (Watchers)

```typescript
watch(() => props.id, (newId) => {
  if (newId) {
    loadData()
  }
}, { immediate: true })

watch(formData, (newData) => {
  console.log('Form data changed:', newData)
}, { deep: true })

watchEffect(() => {
  if (user.value && hasPermission.value) {
    loadUserData()
  }
})
```

### 10. 生命周期钩子 (Lifecycle Hooks)

```typescript
onBeforeMount(() => {
  console.log('Component is about to mount')
})

onMounted(() => {
  loadData()
  initializeComponent()
})

onBeforeUnmount(() => {
  cleanup()
})

onUnmounted(() => {
  console.log('Component unmounted')
})
```

## 🚨 严格规则

### 1. 必须遵循的顺序

1. **类型导入** → **第三方库导入** → **项目内部导入** → **组件导入**
2. **Emits** → **Props** (Emits 必须在 Props 之前)
3. **路由 hooks** → **项目 hooks** → **第三方 hooks**
4. **常量** → **配置对象** → **默认值**
5. **基础响应式变量** → **表单数据** → **列表数据** → **业务数据**
6. **简单计算属性** → **复杂计算属性** → **格式化计算属性**
7. **API 方法** → **事件处理方法** → **业务逻辑方法**
8. **Props 监听** → **数据监听** → **复杂监听**
9. **生命周期钩子按执行顺序**

### 2. 命名规范

- **组件名**: PascalCase (`EvaluateSettings`)
- **文件名**: kebab-case (`evaluate-settings.vue`)
- **变量名**: camelCase (`baseEvaluateInfo`)
- **常量名**: UPPER_SNAKE_CASE (`MAX_ITEMS`)
- **函数名**: camelCase (`addComparisonGroup`)
- **类型名**: PascalCase (`EvaluateInfo`)

### 3. API 调用规范

**🚨 核心原则**：

- **禁止使用 try-catch 包装 useApi**
- **先检查 res，再检查 error**
- **弹窗确认后立即关闭，在目标位置显示加载状态**

```typescript
// ✅ 正确的 API 调用方式
async function loadData() {
  loading.value = true

  const { res, error } = await useApi(someApi, params)

  if (res) {
    data.value = res.data
    ElMessage.success('加载成功')
  }

  if (error) {
    ElMessage.error(error.message || '加载失败')
  }

  loading.value = false
}

// ✅ 正确的弹窗交互流程
function handleConfirm(data: any) {
  dialogVisible.value = false // 立即关闭弹窗
  loadTargetData(data) // 在目标位置加载
}

async function loadTargetData(data: any) {
  targetLoading.value = true // 目标位置显示加载状态

  const { res, error } = await useApi(targetApi, data)

  if (res) {
    targetData.value = res.data
    ElMessage.success('操作成功')
  }

  if (error) {
    ElMessage.error(error.message || '操作失败')
  }

  targetLoading.value = false
}

// ❌ 错误的方式 - 不要使用 try-catch
async function wrongWay() {
  try {
    const { res, error } = await useApi(someApi, params) // 错误！
    // ...
  }
  catch (err) {
    // 错误！useApi 内部已处理异常
  }
}
```

### 4. 注释规范

**🚨 严格禁止无意义注释**：

- **禁止写明显的注释**：如 `
// 4. 组件配置`、`// 5. Emits 定义`、`// 6. 常量和配置`、`// 7. 响应式数据` 等
- **代码应该自解释**：变量名、函数名、类型定义应该清晰表达意图
- **只在复杂业务逻辑处添加必要说明**：如算法逻辑、业务规则、特殊处理等
- **避免重复信息**：不要用注释重复代码已经表达的内容

**正确的注释示例**：

```txt
// ✅ 有意义的注释 - 解释业务逻辑
// Emits 定义必须在 Props 之前，确保类型推导正确
const emit = defineEmits<{...}>()

// 最多只能添加 2 个对比项，符合业务需求
const MAX_ITEMS = 2

// ❌ 无意义的注释 - 代码本身就很清楚
// 响应式数据定义
const loading = ref(false)

// 计算属性
const isValid = computed(() => ...)
```

### 5. 分组和空行规范

- 每个大的代码块之间必须有空行分隔
- 同一类型的变量/方法可以紧挨着写
- 不同功能模块之间用空行分隔，不需要注释

## 完整示例模板

```vue
<script setup lang="ts">
import type { EvaluateDesc, EvaluateInfo } from './evaluate-settings'

import { evaluateI18n } from './evaluate-settings'

import EvaluateSettings from './evaluate-settings.vue'

interface ComparisonItem {
  id: string
  data: EvaluateInfo | null
}

defineOptions({
  name: 'BasicAssessment'
})

const {
  maxItems = 2
} = defineProps<{
  maxItems?: number
}>()

const emit = defineEmits<{
  change: [items: ComparisonItem[]]
}>()

const MAX_ITEMS = 2

const baseEvaluateInfo = ref<EvaluateInfo>({
  title: '示例标题',
  date: '2025-01-01',
  tags: [],
  participantInfo: {
    totalCount: 0,
    targetRatio: 0,
    completionRatio: 0,
  },
  timeRange: ['', '']
})

const comparisonItems = ref<Array<EvaluateInfo | null>>([])

const canAddMore = computed(() => comparisonItems.value.length < MAX_ITEMS)

function addComparisonGroup() {
  if (comparisonItems.value.length < MAX_ITEMS) {
    comparisonItems.value.push(null)
  }
}

function removeComparisonGroup(index: number) {
  comparisonItems.value.splice(index, 1)
}

watch(comparisonItems, (newItems) => {
  emit('change', newItems.map((item, index) => ({
    id: `comparison-${index}`,
    data: item
  })))
}, { deep: true })

onMounted(() => {
  console.log('BasicAssessment mounted')
})
</script>
```

这个规范确保了代码的一致性、可读性和可维护性。
