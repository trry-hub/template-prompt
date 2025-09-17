# 代码规范和命名约定

## 重要规范说明

### API 调用规范
- **useApi 返回值**：`{ res, error }`，两者都可能为 `null`
- **无需 try-catch**：useApi 内部已处理异常，直接检查 `res` 和 `error` 即可
- **错误处理**：通过检查 `error` 对象进行错误处理，不需要额外的异常捕获

### Props 定义规范
- **使用最新语法**：直接使用解构赋值和默认值，不需要 `withDefaults`
- **类型安全**：在 `defineProps<T>()` 中定义完整的 TypeScript 类型
- **默认值**：在解构时直接设置默认值，语法更简洁

### 全局类型规范
- **ApiTypes**：已全局自动引入，无需手动导入，可直接使用
- **其他全局类型**：项目中已配置的全局类型都可直接使用

### API 类型使用规范（🚨 重要）
- **严格按照 api/types 定义**：接口返回值的类型不要重复定义，必须使用 `src/api/types/` 中的类型
- **类型引用方式**：使用 `ApiTypes<typeof api.method>['response']` 获取响应类型
- **数组元素类型**：使用 `ApiTypes<typeof api.method>['response'][number]` 获取数组元素类型
- **composable 类型**：优先使用 composable 中导出的实际类型，如 `GetAcademicSceneListRes`
- **禁止自定义**：不要自己瞎定义接口返回值类型，必须引用已定义的类型
- **类型导入**：尽可能使用引用的方式将 types 类型导入到用到的地方

## 命名规范

### 文件命名
```
组件文件: PascalCase.vue          # UserProfile.vue
页面文件: kebab-case.vue          # user-profile.vue
工具文件: camelCase.ts            # userUtils.ts
类型文件: PascalCase.d.ts         # UserTypes.d.ts
样式文件: kebab-case.scss         # user-profile.scss
```

### 变量命名
```typescript
// 变量和函数: camelCase
const userName = 'John'
const userList = []
function getUserInfo() {}

// 常量: SCREAMING_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_COUNT = 3

// 类型和接口: PascalCase
interface UserInfo {}
interface ApiResponse<T> {}

// 枚举: PascalCase
enum UserStatus {
  Active = 'active',
  Inactive = 'inactive',
}
```

### 组件命名
```typescript
// 组件名: PascalCase
defineOptions({
  name: 'UserProfile',
})

// Props: camelCase
interface Props {
  userId: number
  showAvatar: boolean
  maxCount: number
}

// Emits: camelCase (推荐) 或 kebab-case
const emit = defineEmits<{
  'update:visible': [visible: boolean]
  'userSelected': [user: UserInfo]  // 推荐：camelCase
  'step-change': [step: number]     // 兼容：kebab-case
}>()
```

## Vue 组件规范

### 组件结构顺序
```vue
<script setup lang="ts">
import type { UserInfo } from '@/api/types/user'
// 1. 导入语句
import { computed, onMounted, ref } from 'vue'

// 2. 组件选项
defineOptions({
  name: 'UserProfile',
})

// 3. Props 定义
const {
  userId,
  visible = false
} = defineProps<{
  userId: number
  visible?: boolean
}>()

// 4. Emits 定义
const emit = defineEmits<{
  'update:visible': [visible: boolean]
  'userSelected': [user: UserInfo]
}>()
// 5. 响应式数据
const loading = ref(false)
const userInfo = ref<UserInfo>()

// 6. 计算属性
const displayName = computed(() => {
  return userInfo.value?.name || '未知用户'
})

// 7. 方法定义
async function fetchUserInfo() {
  // 实现...
}

function handleClose() {
  emit('update:visible', false)
}

// 8. 生命周期
onMounted(() => {
  fetchUserInfo()
})
</script>

<template>
  <!-- 模板内容 -->
</template>

<style lang="scss" scoped>
/* 样式内容 */
</style>
```

### Props 规范
```typescript
// ✅ 正确 - 使用最新 Props 语法，直接解构
const {
  userId,
  userName = '',
  visible = false,
  options = []
} = defineProps<{
  userId: number
  userName?: string
  visible?: boolean
  options?: UserOption[]
}>()

// ❌ 错误 - 使用 withDefaults
const props = withDefaults(defineProps<{
  userId: number
  userName?: string
}>(), {
  userName: '',
})

// ❌ 错误 - 使用运行时声明
const props = defineProps({
  userId: Number,
  userName: String,
})

// ✅ 正确 - 使用 API 类型定义 Props
import academicSaasUrls from '@/api/modules/academic-saas'

type SceneOverviewItem = ApiTypes<typeof academicSaasUrls.questInfoOverview>['response'][number]

const {
  sceneOverviewData = []
} = defineProps<{
  sceneOverviewData?: SceneOverviewItem[]
}>()

// ❌ 错误 - 不要自定义接口返回值类型
const {
  sceneData = []
} = defineProps<{
  sceneData?: {  // 不要自定义，应该使用 API 类型
    id: string
    name: string
  }[]
}>()
```

### Emits 规范
```typescript
// ✅ 正确 - 使用 TypeScript 类型，推荐 camelCase
const emit = defineEmits<{
  'update:visible': [visible: boolean]
  'userChange': [user: UserInfo]      // 推荐：camelCase
  'stepChange': [step: number]        // 推荐：camelCase
  'submit': [data: FormData]
}>()

// 触发事件
emit('update:visible', false)
emit('userChange', userInfo.value)
emit('stepChange', 2)

// ❌ 避免 - kebab-case 在 ESLint 中可能报错
const emit = defineEmits<{
  'user-change': [user: UserInfo]     // ESLint 可能要求改为 camelCase
  'step-change': [step: number]       // ESLint 可能要求改为 camelCase
}>()
```

### 对话框组件事件规范
```typescript
// ✅ 推荐的对话框事件命名
const emit = defineEmits<{
  'update:visible': [visible: boolean]  // v-model 绑定
  'confirm': [data: any]                // 确认操作
  'cancel': []                          // 取消操作
  'stepChange': [step: number]          // 步骤变化
  'dataChange': [data: any]             // 数据变化
  'beforeClose': [done: () => void]     // 关闭前回调
}>()

// 使用示例
function handleStepChange() {
  emit('stepChange', stepActive.value)  // ✅ camelCase
}

// ❌ 避免在新代码中使用
function handleStepChange() {
  emit('step-change', stepActive.value) // ESLint 可能报错
}
```

## TypeScript 规范

### 类型定义
```typescript
// 基础类型
interface UserInfo {
  id: number
  name: string
  email: string
  avatar?: string
  status: 'active' | 'inactive'
  createdAt: string
  updatedAt: string
}

// 泛型类型
interface ApiResponse<T> {
  code: number
  data: T
  message: string
}

// 联合类型
type Theme = 'light' | 'dark'
type Size = 'small' | 'medium' | 'large'

// 工具类型
type CreateUserRequest = Omit<UserInfo, 'id' | 'createdAt' | 'updatedAt'>
type UpdateUserRequest = Partial<CreateUserRequest>
```

### 函数类型
```typescript
// 函数签名
type EventHandler<T = any> = (event: T) => void
type AsyncHandler<T, R> = (params: T) => Promise<R>

// 组合函数类型
interface UseUserReturn {
  userInfo: Ref<UserInfo | undefined>
  loading: Ref<boolean>
  error: Ref<string>
  fetchUser: (id: number) => Promise<void>
  updateUser: (data: UpdateUserRequest) => Promise<void>
}
```

## CSS/SCSS 规范

### 类名规范
```scss
// BEM 命名规范 + qxs 前缀
.qxs-user-profile {
  // 块
  &__header {
    // 元素
  }

  &__avatar {
    // 元素
    &--large {
      // 修饰符
    }
  }

  &--loading {
    // 修饰符
  }
}

// 状态类
.is-active {}
.is-disabled {}
.is-loading {}

// 工具类
.text-center {}
.mb-4 {}
.flex-1 {}
```

### 样式组织
```scss
// 1. 变量定义
$primary-color: var(--qxs-color-primary);
$border-radius: var(--qxs-border-radius);

// 2. 混入使用
@include flex-center;
@include text-ellipsis;

// 3. 组件样式
.qxs-user-profile {
  // 布局属性
  display: flex;
  flex-direction: column;

  // 尺寸属性
  width: 100%;
  height: auto;

  // 外观属性
  background-color: var(--qxs-bg-color);
  border: 1px solid var(--qxs-border-color);
  border-radius: $border-radius;

  // 文本属性
  color: var(--qxs-text-color);
  font-size: var(--qxs-font-size);

  // 交互属性
  cursor: pointer;
  transition: all 0.3s ease;

  &:hover {
    background-color: var(--qxs-bg-color-hover);
  }
}
```

## 注释规范

### 函数注释
```typescript
/**
 * 获取用户信息
 * @param userId 用户ID
 * @param includeProfile 是否包含详细信息
 * @returns 用户信息
 */
async function getUserInfo(userId: number, includeProfile = false): Promise<UserInfo> {
  // 实现...
}
```

### 组件注释
```vue
<script setup lang="ts">
/**
 * 用户资料组件
 *
 * 功能:
 * - 显示用户基本信息
 * - 支持编辑用户资料
 * - 支持头像上传
 *
 * @example
 * <UserProfile :user-id="123" @user-change="handleUserChange" />
 */

// 组件实现...
</script>
```

### 复杂逻辑注释
```typescript
// 处理分页数据
function handlePaginationData(data: any[]) {
  // 1. 过滤无效数据
  const validData = data.filter(item => item && item.id)

  // 2. 按创建时间排序
  const sortedData = validData.sort((a, b) =>
    new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
  )

  // 3. 添加序号
  return sortedData.map((item, index) => ({
    ...item,
    index: index + 1,
  }))
}
```

## 错误处理规范

### useApi 调用规范
```typescript
// ✅ 正确 - useApi 无需 try-catch，内部已处理异常
async function fetchData() {
  loading.value = true

  const { res, error } = await useApi(api, params)

  if (res) {
    // 处理成功数据
    dataList.value = res.data
  }

  if (error) {
    ElMessage.error(error.message || '操作失败')
    console.error('API 调用失败:', error)
  }

  loading.value = false
}

// ❌ 错误 - useApi 不需要 try-catch
async function fetchData() {
  try {
    loading.value = true
    const { res, error } = await useApi(api, params) // useApi 内部已处理异常
    // ...
  } catch (err) {
    // 这里的 catch 是多余的
    console.error('Unexpected error:', err)
  } finally {
    loading.value = false
  }
}

// ✅ 特殊情况 - 只有在需要额外逻辑处理时才使用 try-catch
async function complexOperation() {
  loading.value = true

  const { res, error } = await useApi(api, params)

  if (res) {
    // 复杂的数据处理可能抛出异常
    try {
      const processedData = complexDataProcessing(res.data)
      dataList.value = processedData
    } catch (processingError) {
      console.error('数据处理失败:', processingError)
      ElMessage.error('数据处理失败')
    }
  }

  if (error) {
    ElMessage.error(error.message || '操作失败')
  }

  loading.value = false
}
```

### 类型守卫
```typescript
// 类型守卫函数
function isUserInfo(obj: any): obj is UserInfo {
  return obj && typeof obj.id === 'number' && typeof obj.name === 'string'
}

// 使用类型守卫
if (isUserInfo(data)) {
  // 此时 data 被推断为 UserInfo 类型
  console.log(data.name)
}
```

## 性能优化规范

### 响应式数据优化
```typescript
// ✅ 正确 - 使用 shallowRef 优化大对象
const largeData = shallowRef<LargeObject>()

// ✅ 正确 - 使用 markRaw 标记不需要响应式的对象
const chartInstance = markRaw(new Chart())

// ✅ 正确 - 使用 computed 缓存计算结果
const expensiveComputed = computed(() => {
  return heavyCalculation(props.data)
})
```

### 组件优化
```vue
<template>
  <!-- ✅ 正确 - 使用 v-memo 优化列表渲染 -->
  <div v-for="item in list" :key="item.id" v-memo="[item.id, item.status]">
    {{ item.name }}
  </div>

  <!-- ✅ 正确 - 使用 v-once 优化静态内容 -->
  <div v-once>
    {{ staticContent }}
  </div>
</template>
```

## 对话框组件最佳实践

### 数据初始化规范
```typescript
// ✅ 推荐 - 使用 @open 事件初始化
<template>
  <el-dialog
    v-model="visible"
    @open="initializeData"
  >
    <!-- 内容 -->
  </el-dialog>
</template>

<script setup lang="ts">
// ✅ 正确 - 使用 @open 初始化数据
async function initializeData() {
  // 重置状态
  resetState()
  // 获取数据
  await fetchData()
}

// ❌ 避免 - 同时使用 watch 和 @open
watch(visible, (newVisible) => {
  if (newVisible) {
    initializeData() // 与 @open 重复
  }
})
</script>
```

### 事件命名最佳实践
```typescript
// ✅ 推荐的事件命名模式
interface DialogEmits {
  // 基础事件
  'update:visible': [visible: boolean]
  'confirm': [data: any]
  'cancel': []

  // 状态变化事件 - 使用 camelCase
  'stepChange': [step: number]
  'dataChange': [data: any]
  'statusChange': [status: string]

  // 操作事件 - 使用动词形式
  'beforeClose': [done: () => void]
  'afterOpen': []
  'refresh': []
  'reset': []
}

// ESLint 兼容性说明
// - 新项目推荐使用 camelCase
// - 如果 ESLint 要求 camelCase，统一使用 camelCase
// - 避免在同一项目中混用两种命名方式
```

### 加载状态管理
```typescript
// ✅ 统一的加载状态管理 - useApi 无需 try-catch
const loading = ref(false)

async function fetchData() {
  loading.value = true

  const { res, error } = await useApi(api.getData)

  if (res) {
    // 处理成功数据
    dataList.value = res.data
  }

  if (error) {
    ElMessage.error(error.message || '获取数据失败')
    console.error('API 调用失败:', error)
  }

  loading.value = false  // 直接重置，无需 finally
}

// ✅ 多个 API 调用的加载状态管理
async function fetchMultipleData() {
  loading.value = true

  // 并行调用多个 API
  const [result1, result2] = await Promise.all([
    useApi(api.getData1),
    useApi(api.getData2)
  ])

  if (result1.res) {
    data1.value = result1.res
  }
  if (result1.error) {
    ElMessage.error('获取数据1失败')
  }

  if (result2.res) {
    data2.value = result2.res
  }
  if (result2.error) {
    ElMessage.error('获取数据2失败')
  }

  loading.value = false
}
```
