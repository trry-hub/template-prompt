# useApi 使用规范

## 🚨 重要原则：严格按照 api/types 定义使用

> **联调 API 对接接口时，必须完全按照 `src/api/types/` 中定义的返回值类型来写，不要自己瞎定义类型！**

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

### 🎯 类型安全调用（必须遵循）

```typescript
import academicSaasApi from '@/api/modules/academic-saas'

// ✅ 正确 - 使用 ApiTypes 获取 api/types 中定义的类型
const { res, error } = await useApi(academicSaasApi.recommendList, {
  doctorName: 'test',
  pageNo: 1,
  pageSize: 10
})

if (res) {
  // res 的类型自动推断为 api/types/academic-saas.d.ts 中定义的类型
  console.log(res.totalCount) // number
  console.log(res.data)       // 医生列表数组

  // 使用具体的响应数据
  res.data.forEach(doctor => {
    console.log(doctor.name)        // string
    console.log(doctor.hospital)    // string
    console.log(doctor.department)  // string
  })
}

// ❌ 错误 - 不要自己定义返回类型
interface CustomDoctorResponse {  // 🚫 禁止自定义
  list: any[]
  total: number
}
```

### 🔍 如何查找正确的类型定义

#### 1. 查看 API 模块文件
```typescript
// 1. 先查看 src/api/modules/academic-saas.ts
import academicSaasApi from '@/api/modules/academic-saas'

// 2. 找到对应的接口，如：
academicSaasApi.recommendList // 推荐医生列表接口
```

#### 2. 查看对应的类型定义
```typescript
// 3. 查看 src/api/types/academic-saas.d.ts
// 找到对应的类型定义：
export interface AcademicSaasTypes {
  [Urls.recommendList.url]: {
    request: {
      doctorName: string
      pageNo: number
      pageSize: number
    }
    response: {
      totalCount: number
      data: {
        id: string
        name: string
        hospital: string
        department: string
        // ... 更多字段
      }[]
    }
  }
}
```

#### 3. 在代码中使用正确的类型
```typescript
// ✅ 正确 - 直接使用，类型自动推断
const { res, error } = await useApi(academicSaasApi.recommendList, params)

// ✅ 正确 - 显式声明类型（如果需要）
const doctorListData = ref<ApiTypes<typeof academicSaasApi.recommendList>['response']>()

if (res) {
  doctorListData.value = res // 类型完全匹配
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

### 1. 列表数据获取（严格按照 api/types）

```typescript
// ✅ 正确 - 严格按照 api/types 中定义的字段
async function getDoctorList() {
  loading.value = true

  const { res, error } = await useApi(academicSaasApi.recommendList, {
    doctorName: searchForm.value.doctorName,
    pageNo: pagination.value.current,
    pageSize: pagination.value.size
  })

  if (res) {
    // 使用 api/types 中定义的字段
    dataList.value = res.data        // 不是 res.list
    pagination.value.total = res.totalCount  // 不是 res.total
  }

  if (error) {
    ElMessage.error(error.message || '获取医生列表失败')
  }

  loading.value = false
}

// ❌ 错误 - 假设字段名
async function getDataListWrong() {
  const { res, error } = await useApi(academicSaasApi.recommendList, params)
  if (res) {
    dataList.value = res.list    // 🚫 可能不存在 list 字段
    pagination.value.total = res.total  // 🚫 可能是 totalCount 字段
  }
}
```

### 2. 创建/更新操作（严格按照 api/types）

```typescript
// ✅ 正确 - 严格按照 api/types 中定义的请求和响应类型
async function handleSubmit() {
  loading.value = true

  const { res, error } = await useApi(academicSaasApi.createDoctor, {
    // 严格按照 api/types 中定义的 request 类型
    name: formData.value.name,
    hospital: formData.value.hospital,
    department: formData.value.department,
    titleName: formData.value.titleName
  })

  if (res) {
    ElMessage.success('创建成功')
    // 使用 api/types 中定义的响应字段
    if (res.id) {
      router.push(`/doctor/detail/${res.id}`)
    }
  }

  if (error) {
    ElMessage.error(error.message || '创建失败')
  }

  loading.value = false
}

// ❌ 错误 - 使用 any 类型和假设的字段
async function handleSubmitWrong(formData: any) {
  const { res, error } = await useApi(moduleApi.create, formData) // 🚫 any 类型
  if (res) {
    emit('success', res.data) // 🚫 假设有 data 字段
  }
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

## 🚫 常见错误和禁止事项

### 1. 禁止自定义响应类型
```typescript
// ❌ 错误 - 不要自己定义响应类型
interface MyCustomResponse {
  list: any[]
  total: number
  success: boolean
}

// ❌ 错误 - 不要假设字段名
const { res, error } = await useApi(api.getData)
if (res) {
  console.log(res.list)    // 可能不存在 list 字段
  console.log(res.total)   // 可能是 totalCount 字段
}

// ✅ 正确 - 查看 api/types 中的实际定义
if (res) {
  console.log(res.data)       // 实际字段名
  console.log(res.totalCount) // 实际字段名
}
```

### 2. 禁止随意修改字段名
```typescript
// ❌ 错误 - 不要随意转换字段名
if (res) {
  const transformedData = res.data.map(item => ({
    id: item.id,
    title: item.name,      // 🚫 不要随意改名
    content: item.desc     // 🚫 不要随意改名
  }))
}

// ✅ 正确 - 保持原始字段名
if (res) {
  dataList.value = res.data // 直接使用原始数据
}
```

### 3. 禁止忽略错误处理
```typescript
// ❌ 错误 - 忽略错误处理
const { res } = await useApi(api.getData)
// 没有处理 error 的情况

// ✅ 正确 - 完整的错误处理
const { res, error } = await useApi(api.getData)
if (res) {
  // 处理成功
}
if (error) {
  ElMessage.error(error.message || '操作失败')
  console.error('API 调用失败:', error)
}
```

## 📋 API 使用检查清单

在编写 API 调用代码时，请检查：

- [ ] 是否查看了 `src/api/types/` 中对应的类型定义？
- [ ] 是否使用了正确的请求参数类型？
- [ ] 是否使用了正确的响应数据字段？
- [ ] 是否处理了 `error` 情况？
- [ ] 是否添加了适当的加载状态？
- [ ] 是否添加了用户友好的错误提示？
- [ ] 是否避免了自定义响应类型？

## 💡 最佳实践总结

1. **类型优先**：始终以 `api/types` 中的定义为准
2. **字段准确**：使用准确的字段名，不要假设或修改
3. **错误处理**：每个 API 调用都要处理错误情况
4. **用户体验**：提供加载状态和友好的错误提示
5. **代码一致**：团队内保持一致的 API 调用模式
