# useApi 使用规范

## API 调用标准模式

### 基础使用模式

```typescript
// 标准 API 调用
const { res, error } = await useApi(apiConfig, params)

// 处理响应
if (res) {
  // 成功处理
  console.log('成功:', res)
}
if (error) {
  // 错误处理
  ElMessage.error(error.message || '操作失败')
}
```

### 返回值说明

```typescript
interface UseApiResponse<T> {
  res: T | null // 成功时包含响应数据，失败时为 null
  error: { // 失败时包含错误信息，成功时为 null
    code: number
    message: string
    data: null
  } | null
}
```

## 常用场景模式

### 1. 列表数据获取

```typescript
async function getDataList() {
  loading.value = true
  const params = {
    ...getParams(), // 分页参数
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
```

### 2. 创建/更新操作

```typescript
async function handleSubmit(formData: any) {
  loading.value = true
  const apiMethod = isEdit.value ? moduleApi.update : moduleApi.create
  const params = isEdit.value ? { id: currentId.value, ...formData } : formData

  const { res, error } = await useApi(apiMethod, params)
  if (res) {
    ElMessage.success(isEdit.value ? '更新成功' : '创建成功')
    emit('success', res)
    handleClose()
  }
  if (error) {
    ElMessage.error(error.message || '操作失败')
  }
  loading.value = false
}
```

### 3. 删除操作

```typescript
async function handleDelete(item: any) {
  try {
    await ElMessageBox.confirm('确定要删除这个项目吗？', '确认删除', {
      type: 'warning',
    })

    const { res, error } = await useApi(moduleApi.delete, { id: item.id })
    if (res) {
      ElMessage.success('删除成功')
      getDataList() // 刷新列表
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
```

### 4. 批量操作

```typescript
async function handleBatchDelete() {
  if (selectedItems.value.length === 0) {
    ElMessage.warning('请选择要删除的项目')
    return
  }

  try {
    await ElMessageBox.confirm(
      `确定要删除这 ${selectedItems.value.length} 个项目吗？`,
      '批量删除',
      { type: 'warning' }
    )

    const ids = selectedItems.value.map(item => item.id)
    const { res, error } = await useApi(moduleApi.batchDelete, { ids })
    if (res) {
      ElMessage.success('批量删除成功')
      selectedItems.value = []
      getDataList()
    }
    if (error) {
      ElMessage.error(error.message || '批量删除失败')
    }
  }
  catch (err) {
    if (err !== 'cancel') {
      ElMessage.error('批量删除失败')
    }
  }
}
```

### 5. 文件上传

```typescript
async function handleFileUpload(file: File) {
  loading.value = true
  const formData = new FormData()
  formData.append('file', file)

  const { res, error } = await useApi(fileApi.upload, formData)
  if (res) {
    ElMessage.success('上传成功')
    fileUrl.value = res.url
  }
  if (error) {
    ElMessage.error(error.message || '上传失败')
  }
  loading.value = false
}
```

## 错误处理规范

### 统一错误处理

```typescript
// 标准错误处理模式
const { res, error } = await useApi(api, params)
if (error) {
  // 根据错误码进行不同处理
  switch (error.code) {
    case 401:
      ElMessage.error('登录已过期，请重新登录')
      // 跳转到登录页
      break
    case 403:
      ElMessage.error('权限不足')
      break
    case 404:
      ElMessage.error('资源不存在')
      break
    case 500:
      ElMessage.error('服务器错误')
      break
    default:
      ElMessage.error(error.message || '操作失败')
  }
}
```

### 业务错误处理

```typescript
// 业务逻辑错误处理
const { res, error } = await useApi(userApi.login, loginForm)
if (res) {
  // 登录成功
  userStore.setUserInfo(res.userInfo)
  router.push('/')
}
if (error) {
  // 登录失败
  if (error.code === 1001) {
    ElMessage.error('用户名或密码错误')
  }
  else if (error.code === 1002) {
    ElMessage.error('账户已被锁定')
  }
  else {
    ElMessage.error(error.message || '登录失败')
  }
}
```

## 加载状态管理

### 组件级加载状态

```typescript
const loading = ref(false)

async function fetchData() {
  try {
    loading.value = true
    const { res, error } = await useApi(api, params)
    // 处理响应...
  }
  finally {
    loading.value = false
  }
}
```

### 全局加载状态

```typescript
// 使用 store 管理全局加载状态
const appStore = useAppStore()

async function fetchData() {
  appStore.setLoading(true)
  const { res, error } = await useApi(api, params)
  // 处理响应...
  appStore.setLoading(false)
}
```

## 最佳实践

### 1. 始终检查 res 和 error
```typescript
// ✅ 正确
const { res, error } = await useApi(api, params)
if (res) { /* 处理成功 */ }
if (error) { /* 处理错误 */ }

// ❌ 错误
const { res } = await useApi(api, params)
// 没有检查 error
```

### 2. 提供用户友好的错误信息
```typescript
// ✅ 正确
if (error) {
  ElMessage.error(error.message || '获取数据失败')
}

// ❌ 错误
if (error) {
  console.error(error) // 用户看不到错误信息
}
```

### 3. 合理使用 loading 状态
```typescript
// ✅ 正确
try {
  loading.value = true
  const { res, error } = await useApi(api, params)
  // 处理响应...
}
finally {
  loading.value = false // 确保 loading 被重置
}
```

### 4. 避免重复的错误处理代码
```typescript
// ✅ 正确 - 封装通用错误处理
function handleApiError(error: any, defaultMessage = '操作失败') {
  if (error) {
    ElMessage.error(error.message || defaultMessage)
  }
}

const { res, error } = await useApi(api, params)
if (res) { /* 处理成功 */ }
handleApiError(error, '获取数据失败')
```
