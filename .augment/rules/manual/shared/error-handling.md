# 错误处理规范

## 统一错误处理模式

### 基础错误处理结构

```typescript
// 标准错误处理模式
try {
  loading.value = true
  const { res, error } = await useApi(api, params)

  if (res) {
    // 处理成功响应
    handleSuccess(res)
  }

  if (error) {
    // 处理业务错误
    handleBusinessError(error)
  }
}
catch (err) {
  // 处理系统错误
  handleSystemError(err)
}
finally {
  loading.value = false
}
```

## 错误类型分类

### 1. 业务错误（API 返回的错误）
```typescript
interface BusinessError {
  code: number
  message: string
  data: null
}

// 业务错误处理
function handleBusinessError(error: BusinessError) {
  switch (error.code) {
    case 401:
      ElMessage.error('登录已过期，请重新登录')
      // 跳转到登录页
      router.push('/login')
      break
    case 403:
      ElMessage.error('权限不足')
      break
    case 404:
      ElMessage.error('资源不存在')
      break
    case 422:
      ElMessage.error('数据验证失败')
      break
    case 500:
      ElMessage.error('服务器内部错误')
      break
    default:
      ElMessage.error(error.message || '操作失败')
  }
}
```

### 2. 系统错误（网络、代码异常等）
```typescript
// 系统错误处理
function handleSystemError(err: any) {
  console.error('System Error:', err)

  if (err.name === 'NetworkError') {
    ElMessage.error('网络连接失败，请检查网络')
  }
  else if (err.name === 'TimeoutError') {
    ElMessage.error('请求超时，请稍后重试')
  }
  else {
    ElMessage.error('系统错误，请联系管理员')
  }
}
```

### 3. 表单验证错误
```typescript
// 表单验证错误处理
async function validateForm() {
  try {
    await formRef.value?.validate()
    return true
  }
  catch (validationError) {
    // 处理验证错误
    const firstError = Object.values(validationError)[0]
    if (firstError && firstError[0]) {
      ElMessage.error(firstError[0].message)
    }
    return false
  }
}
```

## 常用错误处理模式

### 1. API 调用错误处理
```typescript
// 数据获取错误处理
async function fetchData() {
  try {
    loading.value = true
    error.value = null

    const { res, error: apiError } = await useApi(api.getData, params)

    if (res) {
      data.value = res
    }

    if (apiError) {
      error.value = apiError.message
      ElMessage.error(apiError.message || '获取数据失败')
    }
  }
  catch (err) {
    error.value = '系统错误'
    ElMessage.error('获取数据失败')
  }
  finally {
    loading.value = false
  }
}
```

### 2. 表单提交错误处理
```typescript
// 表单提交错误处理
async function handleSubmit() {
  try {
    // 1. 表单验证
    const isValid = await validateForm()
    if (!isValid) {
      return
    }

    // 2. 提交数据
    submitting.value = true
    const { res, error } = await useApi(api.submit, formData.value)

    if (res) {
      ElMessage.success('提交成功')
      emit('success', res)
    }

    if (error) {
      // 处理业务错误
      if (error.code === 422) {
        // 处理字段验证错误
        handleFieldErrors(error.data)
      }
      else {
        ElMessage.error(error.message || '提交失败')
      }
    }
  }
  catch (err) {
    ElMessage.error('提交失败')
  }
  finally {
    submitting.value = false
  }
}

// 处理字段验证错误
function handleFieldErrors(fieldErrors: Record<string, string[]>) {
  Object.keys(fieldErrors).forEach((field) => {
    const errors = fieldErrors[field]
    if (errors && errors.length > 0) {
      // 设置字段错误状态
      formRef.value?.setFieldError(field, errors[0])
    }
  })
}
```

### 3. 文件上传错误处理
```typescript
// 文件上传错误处理
function handleUploadError(error: any, file: UploadFile) {
  console.error('Upload Error:', error)

  if (error.code === 413) {
    ElMessage.error('文件大小超出限制')
  }
  else if (error.code === 415) {
    ElMessage.error('文件类型不支持')
  }
  else {
    ElMessage.error(error.message || '上传失败')
  }

  // 移除失败的文件
  const fileList = uploadRef.value?.fileList || []
  const index = fileList.findIndex(item => item.uid === file.uid)
  if (index > -1) {
    fileList.splice(index, 1)
  }
}
```

## 全局错误处理

### 1. 全局错误拦截器
```typescript
// 在 main.ts 中设置全局错误处理
app.config.errorHandler = (err, vm, info) => {
  console.error('Global Error:', err, info)

  // 发送错误报告到监控系统
  if (import.meta.env.PROD) {
    reportError(err, info)
  }

  // 显示用户友好的错误信息
  ElMessage.error('应用出现异常，请刷新页面重试')
}

// 未捕获的 Promise 错误
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled Promise Rejection:', event.reason)

  // 阻止默认的控制台错误输出
  event.preventDefault()

  // 显示用户友好的错误信息
  ElMessage.error('操作失败，请稍后重试')
})
```

### 2. 路由错误处理
```typescript
// 路由错误处理
router.onError((error) => {
  console.error('Router Error:', error)

  if (error.message.includes('Loading chunk')) {
    ElMessage.error('页面加载失败，请刷新重试')
  }
  else {
    ElMessage.error('页面跳转失败')
  }
})
```

## 错误边界组件

### 错误边界组件实现
```vue
<script setup lang="ts">
defineOptions({
  name: 'ErrorBoundary',
})

const props = withDefaults(defineProps<Props>(), {
  fallback: '出现了一些问题',
})

interface Props {
  fallback?: string
  onError?: (error: Error, errorInfo: any) => void
}

const hasError = ref(false)
const errorMessage = ref('')

// 捕获子组件错误
onErrorCaptured((error, instance, errorInfo) => {
  console.error('Error Boundary Caught:', error, errorInfo)

  hasError.value = true
  errorMessage.value = error.message

  // 调用错误回调
  props.onError?.(error, errorInfo)

  // 阻止错误继续传播
  return false
})

// 重置错误状态
function resetError() {
  hasError.value = false
  errorMessage.value = ''
}
</script>

<template>
  <div class="error-boundary">
    <div v-if="hasError" class="error-fallback">
      <div class="error-icon">
        ⚠️
      </div>
      <div class="error-message">
        {{ fallback }}
      </div>
      <div class="error-details">
        {{ errorMessage }}
      </div>
      <el-button @click="resetError">
        重试
      </el-button>
    </div>
    <slot v-else />
  </div>
</template>

<style lang="scss" scoped>
.error-fallback {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: var(--qxs-spacing-xl);
  text-align: center;

  .error-icon {
    font-size: 48px;
    margin-bottom: var(--qxs-spacing-md);
  }

  .error-message {
    font-size: var(--qxs-font-size-lg);
    color: var(--qxs-text-color-primary);
    margin-bottom: var(--qxs-spacing-sm);
  }

  .error-details {
    font-size: var(--qxs-font-size-sm);
    color: var(--qxs-text-color-secondary);
    margin-bottom: var(--qxs-spacing-lg);
  }
}
</style>
```

## 错误监控和上报

### 错误上报函数
```typescript
// 错误上报
interface ErrorReport {
  message: string
  stack?: string
  url: string
  userAgent: string
  timestamp: number
  userId?: string
  extra?: any
}

function reportError(error: Error, extra?: any) {
  const report: ErrorReport = {
    message: error.message,
    stack: error.stack,
    url: window.location.href,
    userAgent: navigator.userAgent,
    timestamp: Date.now(),
    userId: getCurrentUserId(),
    extra,
  }

  // 发送到监控系统
  if (import.meta.env.PROD) {
    fetch('/api/errors', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(report),
    }).catch(console.error)
  }
}
```

## 最佳实践

### 1. 错误处理层次
```
1. 组件级错误处理 - 处理具体业务错误
2. 页面级错误边界 - 捕获组件错误
3. 应用级错误处理 - 处理全局错误
4. 系统级错误监控 - 收集和分析错误
```

### 2. 用户体验原则
- **及时反馈**：立即显示错误信息
- **友好提示**：使用用户能理解的语言
- **操作指导**：提供解决方案或重试机制
- **优雅降级**：错误时提供备用方案

### 3. 开发调试原则
- **详细日志**：记录完整的错误信息
- **错误分类**：区分业务错误和系统错误
- **错误追踪**：提供错误堆栈和上下文
- **监控告警**：重要错误及时通知开发团队
