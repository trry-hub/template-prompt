---
type: "agent_requested"
description: "组件开发与优化规范"
---

# Vue 组件开发与优化规范

> 统一的组件开发、拆分、交互逻辑和优化规范

## 组件大小限制

### 基本规则

- **每个 Vue 组件不超过 400 行代码**
- **400 行以内**: 可以写在当前 .vue 文件中，无需拆分
- **超过 400 行**: 必须进行拆分和重构(要保证原有功能不变的前提下)

### 重要说明

- **优先保持单文件**: 如果组件 < 400 行，应该保持在单个 .vue 文件中
- **避免过度拆分**: 不要为了拆分而拆分，简单组件保持单文件更易维护
- **拆分后清理**: 完成开发后必须清理临时文件（测试文件、文档等）
- **自动引入优化**: Vue、Element Plus、全局组件等已自动引入，无需手动引入

### 拆分策略

#### 1. 按内容拆分（UI 组件拆分）

适用于复杂的 UI 结构，如 Tab 切换、多步骤表单等：

```text
components/
├── ParentComponent.vue        # 主组件 (<400 行)
├── ParentComponent/
│   ├── TabContent1.vue       # Tab1 内容组件
│   ├── TabContent2.vue       # Tab2 内容组件
│   ├── TabContent3.vue       # Tab3 内容组件
│   └── StepForm/             # 步骤表单子组件
│       ├── Step1.vue
│       ├── Step2.vue
│       └── Step3.vue
```

**使用场景**:

- Tab 切换中的 content 部分
- 多步骤表单的各个步骤
- 复杂列表的不同展示模式
- 弹窗中的不同内容区域

#### 2. 按页面逻辑拆分（JS 逻辑拆分）

适用于逻辑复杂的组件，将 JS 部分按职责拆分：

```text
components/
├── component-name.vue        # 主组件 (<400 行,不包含样式)
├── component-name/ (主组件不包含样式>400 行进行拆分)
│   ├── index.vue             # 主组件文件(公共逻辑放入这里)
│   ├── interaction.ts     # 前端交互逻辑
│   ├── dataProcess.ts     # 后端交互逻辑
│   ├── constants.ts          # 常量定义
│   └── types.ts              # 类型定义
```

**逻辑拆分原则**:

- **前端交互逻辑** (`interaction.ts`): 表单验证、UI 状态管理、用户交互处理
- **后端交互逻辑** (`dataProcess.ts`): API 调用、数据获取、状态同步
- **公共逻辑** (`index.vue`): 通用业务逻辑、数据处理、工具方法

#### 3. 拆分决策流程（重要：按优先级执行）

##### 第一步：优先使用逻辑拆分
**默认策略**：超过400行的组件应该**优先考虑逻辑拆分**

- **逻辑拆分适用场景**：
  - 组件包含大量业务逻辑（>100行JS逻辑）
  - 有多个API调用和数据处理
  - 表单验证、状态管理复杂
  - 事件处理和交互逻辑复杂

##### 第二步：特殊情况使用内容拆分
**仅在以下情况使用内容拆分**：

- **页面内容极其庞大**（>800行，且UI结构占主要部分）
- **明确的独立功能模块**（如Tab页面、多步骤向导）
- **可复用的独立UI组件**（如卡片、列表项）
- **逻辑拆分后仍然超过400行**

##### 第三步：拆分执行原则

**逻辑拆分执行步骤**：

1. 创建 `component-name/` 目录（使用中划线命名）
2. 将主组件重命名为 `component-name/index.vue`
3. 按职责创建组合函数：
   - `interaction.ts` - 前端交互逻辑
   - `dataProcess.ts` - 后端数据处理
   - `constants.ts` - 常量定义
   - `types.ts` - 类型定义
4. 主组件只保留模板和组合函数调用

**内容拆分执行步骤**：

1. 创建 `component-name/` 目录（使用中划线命名）
2. 按功能模块拆分子组件
3. 主组件保持数据流和事件处理
4. 子组件专注于UI展示和局部交互

##### 第四步：拆分质量检查
- 主组件代码 < 400行
- 逻辑清晰，职责分离
- **严格保持原有功能完整性**
- **严格保持原有样式不变**
- **严格保持原有逻辑不变**
- 组件间通信简洁明了

#### 4. 拆分核心原则（🚨 极其重要）

##### 保持原有代码不变原则
- **样式完全保持**：所有CSS样式必须完全保持不变
- **逻辑完全保持**：所有业务逻辑必须完全保持不变
- **功能完全保持**：所有功能行为必须完全保持不变
- **数据流保持**：所有数据流向必须完全保持不变
- **事件处理保持**：所有事件处理必须完全保持不变

##### 拆分方式限制
- **仅移动代码**：只能移动代码到新文件，不能修改代码逻辑
- **仅提取函数**：只能将代码块提取为函数，不能改变执行逻辑
- **保持导入导出**：保持所有原有的导入导出关系
- **保持变量名**：保持所有原有的变量名和方法名
- **保持计算属性**：保持所有原有的计算属性逻辑

##### 拆分验证标准
- **功能测试**：拆分后所有功能必须与拆分前完全一致
- **样式测试**：拆分后所有样式必须与拆分前完全一致
- **交互测试**：拆分后所有交互必须与拆分前完全一致
- **数据测试**：拆分后所有数据处理必须与拆分前完全一致

---

## 🎯 组件交互逻辑规范

### 适用场景

当需要开发以下功能时，必须严格按照此规范执行：

- 动态添加/删除组件或数据项
- 表单交互和数据绑定
- 列表操作（增删改查）
- 弹窗和对话框交互
- 状态管理和数据流控制

### 🚨 核心原则

#### 1. 数据驱动视图
- 所有 UI 状态都由响应式数据控制
- 不直接操作 DOM，通过数据变化驱动视图更新
- 使用计算属性处理派生状态

#### 2. 单向数据流
- 父组件向子组件传递数据通过 props
- 子组件向父组件传递数据通过 emit 事件
- 兄弟组件通信通过共同的父组件或状态管理

#### 3. 职责分离
- 数据管理逻辑与视图渲染逻辑分离
- 业务逻辑与 UI 交互逻辑分离
- 每个方法只负责一个具体功能

### 📋 交互逻辑开发模板

#### 动态列表管理模板

```vue
<script setup lang="ts">
// 1. 类型定义
interface ListItem {
  id: string
  data: any | null
  status?: 'active' | 'inactive'
}

// 2. 组件配置
defineOptions({
  name: 'DynamicList'
})

const {
  maxItems = 10,
  allowEmpty = true
} = defineProps<{
  maxItems?: number
  allowEmpty?: boolean
}>()

// 3. Emits 和 Props
const emit = defineEmits<{
  change: [items: ListItem[]]
  add: [item: ListItem]
  remove: [id: string, index: number]
}>()

// 4. 常量配置
const DEFAULT_ITEM: Partial<ListItem> = {
  data: null,
  status: 'active'
}

// 5. 响应式数据
const items = ref<ListItem[]>([])
const loading = ref(false)

// 6. 计算属性
const canAddMore = computed(() => items.value.length < maxItems)
const hasItems = computed(() => items.value.length > 0)
const activeItems = computed(() => items.value.filter(item => item.status === 'active'))

// 7. 核心方法
function generateId(): string {
  return `item-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`
}

function addItem(data: any = null): void {
  if (!canAddMore.value) {
    ElMessage.warning(`最多只能添加 ${maxItems} 个项目`)
    return
  }

  const newItem: ListItem = {
    id: generateId(),
    data,
    status: 'active'
  }

  items.value.push(newItem)
  emit('add', newItem)
}

function removeItem(index: number): void {
  if (index < 0 || index >= items.value.length) {
    console.warn('Invalid index for removeItem:', index)
    return
  }

  const item = items.value[index]
  items.value.splice(index, 1)
  emit('remove', item.id, index)
}

function updateItem(index: number, data: any): void {
  if (index < 0 || index >= items.value.length) {
    console.warn('Invalid index for updateItem:', index)
    return
  }

  items.value[index].data = data
}

// 8. 监听器
watch(items, (newItems) => {
  emit('change', newItems)
}, { deep: true })

// 9. 生命周期
onMounted(() => {
  // 初始化逻辑
})
</script>
```

### 🚨 必须遵循的交互规则

1. **数据初始化**：所有响应式数据必须有明确的初始值
2. **类型安全**：所有方法参数和返回值必须有类型定义
3. **错误处理**：所有异步操作必须有错误处理
4. **边界检查**：数组操作必须检查索引边界
5. **用户反馈**：重要操作必须有用户反馈（成功/失败提示）
6. **加载状态**：异步操作必须有加载状态管理
7. **数据验证**：表单数据必须有验证逻辑
8. **事件通信**：组件间通信必须通过规范的 emit 事件

---

## 代码书写顺序规范

详细的代码书写顺序规范请参考：[代码书写顺序规范](./code-writing-order.md)

### 核心原则

1. **引入按依赖强度排序**: Vue 核心 → UI 框架 → 项目组合函数 → API → 组件
2. **代码块按逻辑顺序**: 类型定义 → 配置 → Props/Emits → Hooks → 变量 → 计算属性 → 监听器 → 方法 → 生命周期
3. **生命周期按执行顺序**: onBeforeMount → onMounted → onBeforeUpdate → onUpdated → onBeforeUnmount → onUnmounted

### 拆分时机判断

#### 何时使用内容拆分

- Tab 切换内容复杂（每个 Tab > 100 行）
- 多步骤表单（每个步骤 > 80 行）
- 列表有多种展示模式（卡片、表格、详情等）
- 弹窗内容复杂且可复用

#### 何时使用逻辑拆分

- API 调用逻辑复杂（> 100 行）
- 表单验证逻辑复杂（> 80 行）
- 业务逻辑复杂且可复用
- 交互状态管理复杂

## 组合函数规范

### 组合函数拆分原则

- **前端交互逻辑** (`interaction.ts`): 表单验证、UI 状态管理、用户交互处理
- **后端交互逻辑** (`useDataProces.ts`): API 调用、数据获取、状态同步
- **公共逻辑** (`useCommon.ts`): 通用业务逻辑、数据处理、工具方法

### 文件职责分离

- **utils.ts**: 纯函数工具，无副作用
- **constants.ts**: 静态配置数据
- **types.ts**: TypeScript 类型定义
- **styles.scss**: 组件专用样式

## 代码组织规范

### 主组件结构

```vue
<script setup lang="ts">
import { CONSTANTS } from './component-name/constants'
import { useDataProcess } from './component-name/dataProcess'
// 1. 导入组合函数和工具
import { useInteraction } from './component-name/interaction'

// 2. 组件配置
defineOptions({ name: 'ComponentName' })

// 3. Props 和 Emits
const props = defineProps<PropsType>()
const emit = defineEmits<EmitsType>()

// 4. 响应式数据
const localState = ref()

// 5. 使用组合函数
const { formData, loading, handleSubmit } = useInteraction()
const { apiData, loadData } = useDataProcess()

// 6. 生命周期
onMounted(() => {
  loadData()
})
</script>

<template>
  <!-- 模板内容 -->
</template>

<style lang="scss" scoped>
// 样式内容
</style>
```

### 组合函数规范

```typescript
// useFeatureName.ts
export function useFeatureName(params: ParamsType) {
  // 1. 响应式状态
  const state = ref()

  // 2. 计算属性
  const computed = computed(() => {})

  // 3. 方法定义
  function method() {}

  // 4. 副作用处理
  watch(state, () => {})

  // 5. 返回公共接口
  return {
    state,
    computed,
    method,
  }
}
```

## 性能优化规范

### 1. 计算属性优化

- 复杂计算逻辑提取到组合函数
- 避免在模板中进行复杂计算
- 使用 `shallowRef` 和 `shallowReactive` 优化大对象

### 2. 事件处理优化

- 统一的事件处理函数
- 避免在模板中定义内联函数
- 使用防抖和节流优化频繁操作

### 3. 组件懒加载

- 大型组件使用 `defineAsyncComponent`
- 路由级别的代码分割
- 条件渲染的组件懒加载

## 代码质量要求

### 1. TypeScript 类型安全

- 所有 props、emits、返回值必须有类型定义
- 使用 API 类型定义，避免重复定义
- 组合函数必须有完整的类型注解

### 2. 错误处理

- API 调用必须有错误处理
- 用户友好的错误提示
- 边界情况的容错处理

### 3. 可维护性

- 清晰的函数命名和注释
- 单一职责原则
- 易于测试的代码结构

## 开发完成后清理规则

### 必须清理的临时文件

- **测试组件**: `*-test.vue`、`*-demo.vue` 等测试文件
- **临时文档**: `*-summary.md`、`*-notes.md` 等开发过程文档
- **调试文件**: `*-debug.vue`、`*-temp.vue` 等调试文件
- **示例文件**: `*-example.vue`、`*-sample.vue` 等示例文件

### 保留的文件

- **主组件**: 实际使用的 .vue 组件文件
- **正式文档**: README.md、API.md 等正式文档
- **配置文件**: 必要的配置和常量文件

### 清理检查清单

- [ ] 删除所有测试组件文件
- [ ] 删除所有临时文档文件
- [ ] 删除所有调试和示例文件
- [ ] 保留必要的正式文件
- [ ] 确认组件功能正常

## 重构检查清单

### 组件拆分检查

- [ ] 组件代码行数 < 400 行
- [ ] 逻辑按功能模块分离（仅当 > 400 行时）
- [ ] 避免过度拆分
- [ ] 类型定义完整

### 性能检查

- [ ] 无不必要的重复渲染
- [ ] 计算属性使用合理
- [ ] 事件处理优化
- [ ] 内存泄漏检查

### 代码质量检查

- [ ] TypeScript 类型安全
- [ ] 错误处理完善
- [ ] 代码可读性良好
- [ ] 符合项目规范

### 完成后清理检查

- [ ] 已删除所有临时测试文件
- [ ] 已删除所有开发过程文档
- [ ] 只保留必要的正式文件
- [ ] 组件功能验证通过

## 实际应用示例

### 示例 1: Tab 切换组件（内容拆分）

```vue
<!-- user-management.vue (主组件) -->
<script setup lang="ts">
import UserList from './user-management/UserList.vue'
import UserLog from './user-management/UserLog.vue'
import UserPermission from './user-management/UserPermission.vue'

const activeTab = ref('list')
const selectedUserId = ref('')

function handleEdit(userId: string) {
  selectedUserId.value = userId
  activeTab.value = 'permission'
}
</script>

<template>
  <div class="user-management">
    <el-tabs v-model="activeTab">
      <el-tab-pane label="用户列表" name="list">
        <UserList @edit="handleEdit" />
      </el-tab-pane>
      <el-tab-pane label="权限管理" name="permission">
        <UserPermission :user-id="selectedUserId" />
      </el-tab-pane>
      <el-tab-pane label="操作日志" name="log">
        <UserLog :user-id="selectedUserId" />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
```

### 示例 2: 复杂表单组件（逻辑拆分）

```vue
<!-- product-form/index.vue (主组件) -->
<script setup lang="ts">
import { useDataProcess } from './dataProcess'
import { useInteraction } from './interaction'
import { useCommon } from './useCommon'

// 前端交互逻辑
const {
  formRef,
  formData,
  rules,
  validateForm,
  resetForm
} = useInteraction()

// 后端交互逻辑
const {
  loading,
  submitForm,
  loadProductData
} = useDataProcess(formData)

async function handleSubmit() {
  const isValid = await validateForm()
  if (isValid) {
    await submitForm()
  }
}

onMounted(() => {
  loadProductData()
})
</script>
```
