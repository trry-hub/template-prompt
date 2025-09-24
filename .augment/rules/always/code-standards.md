---
type: "always_apply"
description: "代码规范和命名约定"
---

# 代码规范和命名约定

## 重要规范说明

### API 调用规范（🚨 极其重要）
- **useApi 返回值**：`{ res, error }`，两者都可能为 `null`
- **严禁使用 try-catch**：useApi 内部已处理异常，直接检查 `res` 和 `error` 即可
- **错误处理模式**：通过检查 `error` 对象进行错误处理，不需要额外的异常捕获

#### 正确的 API 调用方式

```typescript
// ✅ 正确：直接使用 useApi 返回值
async function fetchData() {
  loading.value = true

  const { res, error } = await useApi(apiUrls.getData, params)
  if (res) {
    // 处理成功响应
    data.value = res.data
  }
  if (error) {
    // 处理错误
    ElMessage.error(error.message || '请求失败')
  }

  loading.value = false
}
```

#### 错误的 API 调用方式

```typescript
// ❌ 错误：不要使用 try-catch 包装 useApi
async function fetchData() {
  try {
    loading.value = true
    const { res, error } = await useApi(apiUrls.getData, params)
    // ... 处理逻辑
  } catch (error) {
    // 这是多余的，useApi 内部已处理异常
    console.error(error)
  } finally {
    loading.value = false
  }
}
```

### Props 定义规范

- **使用最新语法**：直接使用解构赋值和默认值，不需要 `withDefaults`
- **类型安全**：在 `defineProps<T>()` 中定义完整的 TypeScript 类型
- **默认值**：在解构时直接设置默认值，语法更简洁

### 全局类型规范

- **ApiTypes**：已全局自动引入，无需手动导入，可直接使用
- **其他全局类型**：项目中已配置的全局类型都可直接使用

### API 类型使用规范（🚨 重要）

- **严格按照 api/types 定义**：接口返回值的类型不要重复定义，必须使用 `src/api/types/` 中的类型

## 🚨 特别强调：useApi 调用规范

### 核心原则
1. **useApi 内部已处理所有异常**，返回 `{ res, error }` 结构
2. **严禁在 useApi 外层使用 try-catch**，这是多余且错误的做法
3. **直接检查 res 和 error** 进行业务逻辑处理

### 标准调用模式
```typescript
// ✅ 正确的调用方式
async function apiCall() {
  loading.value = true

  const { res, error } = await useApi(apiUrls.getData, params)
  if (res) {
    // 处理成功响应
    data.value = res.data
  }
  if (error) {
    // 处理错误
    ElMessage.error(error.message || '请求失败')
  }

  loading.value = false
}
```

### 禁止的错误用法
```typescript
// ❌ 错误：不要用 try-catch 包装 useApi
async function wrongApiCall() {
  try {
    const { res, error } = await useApi(apiUrls.getData, params)
    // 这样做是错误的！
  } catch (error) {
    // useApi 内部已处理异常，这里永远不会执行
  }
}
```

### 特殊情况处理
对于非 useApi 的异步操作（如 ElMessageBox.confirm），可以使用 `.catch()` 方法：
```typescript
// ✅ 正确：对于非 useApi 的异步操作
const confirmResult = await ElMessageBox.confirm('确认删除？', '提示', {
  type: 'warning'
}).catch(() => 'cancel')

if (confirmResult === 'cancel') {
  return
}

// 然后调用 useApi（不使用 try-catch）
const { res, error } = await useApi(apiUrls.delete, { id })
if (res) {
  ElMessage.success('删除成功')
}
if (error) {
  ElMessage.error(error.message || '删除失败')
}
```
