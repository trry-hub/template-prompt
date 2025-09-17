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

// Emits: kebab-case
const emit = defineEmits<{
  'update:visible': [visible: boolean]
  'user-selected': [user: UserInfo]
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
  'user-selected': [user: UserInfo]
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
// ✅ 正确 - 使用最新 Props 语法
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

// ❌ 错误 - 使用运行时声明
const props = defineProps({
  userId: Number,
  userName: String,
})
```

### Emits 规范
```typescript
// ✅ 正确 - 使用 TypeScript 类型
const emit = defineEmits<{
  'update:visible': [visible: boolean]
  'user-change': [user: UserInfo]
  'submit': [data: FormData]
}>()

// 触发事件
emit('update:visible', false)
emit('user-change', userInfo.value)
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

### Try-Catch 使用
```typescript
// ✅ 正确
async function fetchData() {
  try {
    loading.value = true
    const { res, error } = await useApi(api, params)
    if (res) {
      // 处理成功
    }
    if (error) {
      ElMessage.error(error.message || '操作失败')
    }
  }
  catch (err) {
    console.error('Unexpected error:', err)
    ElMessage.error('系统错误')
  }
  finally {
    loading.value = false
  }
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
