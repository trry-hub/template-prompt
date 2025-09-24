---
type: "agent_requested"
description: "usePagination 使用规范"
---

# usePagination 使用规范

## 分页管理标准模式

### 基础使用

```typescript
// 从 @qxs-bns/hooks 自动引入
const { pagination, getParams, onSizeChange, onCurrentChange, onSortChange } = usePagination()
```

### 返回值说明

```typescript
interface PaginationReturn {
  pagination: {
    page: number // 当前页码
    size: number // 每页条数
    total: number // 总条数
    sizes: number[] // 每页条数选项 [10, 20, 50, 100]
    layout: string // 分页组件布局
  }
  getParams: () => { // 获取分页参数
    page: number
    size: number
  }
  onSizeChange: (size: number) => Promise<void> // 每页条数变化
  onCurrentChange: (page: number) => Promise<void> // 当前页变化
  onSortChange: (prop: string, order: string) => Promise<void> // 排序变化
}
```

## 标准使用模式

### 1. 列表页面分页

```vue
<script setup lang="ts">
// 分页管理
const { pagination, getParams, onSizeChange, onCurrentChange, onSortChange } = usePagination()
const loading = ref(false)
const dataList = ref([])

// 搜索参数
const searchParams = ref({
  keyword: '',
  status: '',
  dateRange: [],
})

// 获取数据列表
async function getDataList() {
  loading.value = true
  const params = {
    ...getParams(), // 分页参数 { page, size }
    ...searchParams.value, // 搜索参数
  }

  const { res, error } = await useApi(moduleApi.list, params)
  if (res) {
    dataList.value = res.list
    pagination.value.total = res.total
  }
  if (error) {
    ElMessage.error(error.message || '获取数据失败')
  }
  loading.value = false
}

// 搜索处理
function handleSearch(params: any) {
  searchParams.value = params
  onCurrentChange(1).then(() => getDataList())
}

function handleReset() {
  searchParams.value = { keyword: '', status: '', dateRange: [] }
  onCurrentChange(1).then(() => getDataList())
}

// 分页事件处理
function handleSizeChange(size: number) {
  onSizeChange(size).then(() => getDataList())
}

function handleCurrentChange(page: number) {
  onCurrentChange(page).then(() => getDataList())
}

function handleSortChange({ prop, order }: { prop: string, order: string }) {
  onSortChange(prop, order).then(() => getDataList())
}

// 初始化
onMounted(() => {
  getDataList()
})
</script>

<template>
  <!-- 数据表格 -->
  <el-table
    v-loading="loading"
    :data="dataList"
    @sort-change="handleSortChange"
  >
    <!-- 表格列... -->
  </el-table>

  <!-- 分页组件 -->
  <div class="pagination-wrapper">
    <el-pagination
      :current-page="pagination.page"
      :total="pagination.total"
      :page-size="pagination.size"
      :page-sizes="pagination.sizes"
      :layout="pagination.layout"
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>
```

### 2. 表格排序集成

```typescript
// 表格排序处理
function handleSortChange({ prop, order }: { prop: string, order: string }) {
  // onSortChange 会自动更新内部排序状态
  onSortChange(prop, order).then(() => {
    // 重新获取数据
    getDataList()
  })
}

// 在 getParams() 中会自动包含排序参数
const params = {
  ...getParams(), // 包含 { page, size, sortBy, sortOrder }
  ...searchParams.value,
}
```

### 3. 搜索重置分页

```typescript
// 搜索时重置到第一页
function handleSearch(searchForm: any) {
  searchParams.value = { ...searchForm }
  // 重置到第一页并获取数据
  onCurrentChange(1).then(() => getDataList())
}

// 重置搜索条件
function handleReset() {
  searchParams.value = {
    keyword: '',
    status: '',
    dateRange: [],
  }
  // 重置到第一页并获取数据
  onCurrentChange(1).then(() => getDataList())
}
```

### 4. 操作后刷新当前页

```typescript
// 删除操作后刷新当前页
async function handleDelete(item: any) {
  try {
    await ElMessageBox.confirm('确定要删除这个项目吗？', '确认删除', {
      type: 'warning',
    })

    const { res, error } = await useApi(moduleApi.delete, { id: item.id })
    if (res) {
      ElMessage.success('删除成功')
      // 刷新当前页数据
      getDataList()
    }
    if (error) {
      ElMessage.error(error.message || '删除失败')
    }
  }
  catch (err) {
    if (err !== 'cancel') {
      ElMessage.error('删除失败')
    }
  }
}

// 批量删除后刷新
async function handleBatchDelete() {
  // ... 批量删除逻辑
  if (res) {
    ElMessage.success('批量删除成功')
    selectedItems.value = []
    // 如果当前页没有数据了，回到上一页
    if (dataList.value.length <= selectedItems.value.length && pagination.page > 1) {
      onCurrentChange(pagination.page - 1).then(() => getDataList())
    }
    else {
      getDataList()
    }
  }
}
```

## 高级用法

### 1. 自定义分页配置

```typescript
// 如果需要自定义分页配置，可以在初始化后修改
const { pagination, getParams, onSizeChange, onCurrentChange } = usePagination()

// 自定义每页条数选项
pagination.value.sizes = [5, 10, 20, 50]
// 自定义布局
pagination.value.layout = 'total, prev, pager, next, sizes'
```

### 2. 监听分页变化

```typescript
// 监听分页参数变化
watch(
  () => [pagination.page, pagination.size],
  ([newPage, newSize], [oldPage, oldSize]) => {
    if (newPage !== oldPage || newSize !== oldSize) {
      console.log('分页参数变化:', { page: newPage, size: newSize })
    }
  }
)
```

### 3. 分页状态持久化

```typescript
// 如果需要在路由切换时保持分页状态
const route = useRoute()
const router = useRouter()

// 更新 URL 查询参数
function updateUrlParams() {
  const query = {
    ...route.query,
    page: pagination.page.toString(),
    size: pagination.size.toString(),
  }
  router.replace({ query })
}

// 从 URL 恢复分页状态
function restorePaginationFromUrl() {
  const { page, size } = route.query
  if (page) {
    pagination.page = Number(page)
  }
  if (size) {
    pagination.size = Number(size)
  }
}

onMounted(() => {
  restorePaginationFromUrl()
  getDataList()
})
```

## 最佳实践

### 1. 始终使用 getParams() 获取分页参数
```typescript
// ✅ 正确
const params = {
  ...getParams(),
  ...searchParams.value,
}

// ❌ 错误
const params = {
  page: pagination.page,
  size: pagination.size,
  ...searchParams.value,
}
```

### 2. 使用 Promise 链式调用
```typescript
// ✅ 正确
onCurrentChange(1).then(() => getDataList())

// ❌ 错误
pagination.page = 1
getDataList() // 可能获取到旧的分页参数
```

### 3. 合理处理分页边界情况
```typescript
// 删除最后一页的最后一条数据时，回到上一页
if (dataList.value.length === 1 && pagination.page > 1) {
  onCurrentChange(pagination.page - 1).then(() => getDataList())
}
else {
  getDataList()
}
```

### 4. 分页组件样式统一
```vue
<template>
  <div class="pagination-wrapper">
    <el-pagination
      :current-page="pagination.page"
      :total="pagination.total"
      :page-size="pagination.size"
      :page-sizes="pagination.sizes"
      :layout="pagination.layout"
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<style lang="scss" scoped>
.pagination-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  padding: 20px 0;
}
</style>
```
