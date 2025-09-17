# 权限控制规范

## 权限控制组件使用

### Auth 组件基础使用

```vue
<template>
  <!-- 单个权限控制 -->
  <Auth auth="user:create">
    <el-button type="primary">
      新增用户
    </el-button>
  </Auth>

  <!-- 多个权限控制（OR 关系） -->
  <Auth :auth="['user:edit', 'user:delete']">
    <el-button>编辑或删除</el-button>
  </Auth>

  <!-- 多个权限控制（AND 关系） -->
  <AuthAll :auth="['user:edit', 'admin:manage']">
    <el-button>管理员编辑</el-button>
  </AuthAll>
</template>
```

### 权限字符串规范

```typescript
// 权限命名规范：模块:操作
'user:create' // 用户创建
'user:edit' // 用户编辑
'user:delete' // 用户删除
'user:view' // 用户查看
'user:list' // 用户列表

'role:create' // 角色创建
'role:edit' // 角色编辑
'role:delete' // 角色删除
'role:assign' // 角色分配

'system:config' // 系统配置
'system:log' // 系统日志
'system:backup' // 系统备份
```

## 权限检查 Composable

### useAuth 使用模式

```vue
<script setup lang="ts">
// 权限检查 Composable
const { hasPermission, hasAnyPermission, hasAllPermissions, userInfo } = useAuth()

// 单个权限检查
const canCreate = computed(() => hasPermission('user:create'))
const canEdit = computed(() => hasPermission('user:edit'))
const canDelete = computed(() => hasPermission('user:delete'))

// 多个权限检查（OR 关系）
const canManage = computed(() => hasAnyPermission(['user:edit', 'user:delete']))

// 多个权限检查（AND 关系）
const canAdminManage = computed(() => hasAllPermissions(['user:edit', 'admin:manage']))

// 在方法中使用权限检查
function handleEdit() {
  if (!hasPermission('user:edit')) {
    ElMessage.warning('权限不足')
  }
  // 执行编辑操作
}

function handleDelete() {
  if (!hasPermission('user:delete')) {
    ElMessage.warning('权限不足')
  }
  // 执行删除操作
}
</script>

<template>
  <!-- 在模板中使用权限检查 -->
  <div class="actions">
    <el-button v-if="canCreate" type="primary" @click="handleCreate">
      新增
    </el-button>

    <el-button v-if="canEdit" @click="handleEdit">
      编辑
    </el-button>

    <el-button v-if="canDelete" type="danger" @click="handleDelete">
      删除
    </el-button>
  </div>
</template>
```

## 路由权限控制

### 路由守卫权限检查

```typescript
// router/guards.ts
import { useAuth } from '@/composables/useAuth'

// 路由权限检查
router.beforeEach(async (to, from, next) => {
  const { isAuthenticated, hasPermission, getUserInfo } = useAuth()

  // 检查是否需要登录
  if (to.meta.requiresAuth && !isAuthenticated.value) {
    next('/login')
    return
  }

  // 检查路由权限
  if (to.meta.permission) {
    // 确保用户信息已加载
    if (!getUserInfo()) {
      try {
        await loadUserInfo()
      }
      catch (error) {
        next('/login')
        return
      }
    }

    // 检查权限
    const permission = to.meta.permission as string
    if (!hasPermission(permission)) {
      next('/403') // 跳转到无权限页面
      return
    }
  }

  next()
})
```

### 路由权限配置

```typescript
// router/modules/user.ts
export default [
  {
    path: '/user',
    name: 'User',
    component: () => import('@/views/user/index.vue'),
    meta: {
      title: '用户管理',
      requiresAuth: true,
      permission: 'user:list', // 需要的权限
    },
  },
  {
    path: '/user/create',
    name: 'UserCreate',
    component: () => import('@/views/user/create.vue'),
    meta: {
      title: '新增用户',
      requiresAuth: true,
      permission: 'user:create',
    },
  },
  {
    path: '/user/edit/:id',
    name: 'UserEdit',
    component: () => import('@/views/user/edit.vue'),
    meta: {
      title: '编辑用户',
      requiresAuth: true,
      permission: 'user:edit',
    },
  },
] as RouteRecordRaw[]
```

## 菜单权限控制

### 动态菜单生成

```typescript
// composables/useMenu.ts
export function useMenu() {
  const { hasPermission } = useAuth()

  // 原始菜单配置
  const rawMenus = [
    {
      id: 'user',
      title: '用户管理',
      icon: 'user',
      permission: 'user:list',
      children: [
        {
          id: 'user-list',
          title: '用户列表',
          path: '/user',
          permission: 'user:list',
        },
        {
          id: 'user-create',
          title: '新增用户',
          path: '/user/create',
          permission: 'user:create',
        },
      ],
    },
    {
      id: 'role',
      title: '角色管理',
      icon: 'role',
      permission: 'role:list',
      children: [
        {
          id: 'role-list',
          title: '角色列表',
          path: '/role',
          permission: 'role:list',
        },
      ],
    },
  ]

  // 过滤有权限的菜单
  const filterMenus = (menus: any[]): any[] => {
    return menus.filter((menu) => {
      // 检查菜单权限
      if (menu.permission && !hasPermission(menu.permission)) {
        return false
      }

      // 递归过滤子菜单
      if (menu.children) {
        menu.children = filterMenus(menu.children)
        // 如果子菜单为空，隐藏父菜单
        return menu.children.length > 0
      }

      return true
    })
  }

  // 可访问的菜单
  const accessibleMenus = computed(() => filterMenus(rawMenus))

  return {
    accessibleMenus,
  }
}
```

## 数据权限控制

### 数据过滤权限

```typescript
// 数据权限过滤
function useDataPermission() {
  const { userInfo, hasPermission } = useAuth()

  // 根据用户权限过滤数据
  function filterDataByPermission<T>(data: T[], permission: string): T[] {
    if (!hasPermission(permission)) {
      return []
    }

    // 根据用户角色过滤数据
    if (userInfo.value?.role === 'admin') {
      return data // 管理员可以看到所有数据
    }

    if (userInfo.value?.role === 'manager') {
      // 管理者只能看到自己部门的数据
      return data.filter((item: any) =>
        item.departmentId === userInfo.value?.departmentId
      )
    }

    // 普通用户只能看到自己的数据
    return data.filter((item: any) =>
      item.userId === userInfo.value?.id
    )
  }

  // 检查是否可以操作某条数据
  function canOperateData(data: any, operation: string): boolean {
    const permission = `${data.module}:${operation}`

    if (!hasPermission(permission)) {
      return false
    }

    // 检查数据所有权
    if (userInfo.value?.role === 'admin') {
      return true
    }

    if (userInfo.value?.role === 'manager') {
      return data.departmentId === userInfo.value?.departmentId
    }

    return data.userId === userInfo.value?.id
  }

  return {
    filterDataByPermission,
    canOperateData,
  }
}
```

## 权限状态管理

### Pinia 权限 Store

```typescript
// stores/auth.ts
export const useAuthStore = defineStore('auth', () => {
  // 状态
  const token = ref<string>('')
  const userInfo = ref<UserInfo | null>(null)
  const permissions = ref<string[]>([])
  const roles = ref<string[]>([])

  // 计算属性
  const isAuthenticated = computed(() => !!token.value)

  // 权限检查方法
  function hasPermission(permission: string): boolean {
    return permissions.value.includes(permission)
  }

  function hasAnyPermission(perms: string[]): boolean {
    return perms.some(perm => permissions.value.includes(perm))
  }

  function hasAllPermissions(perms: string[]): boolean {
    return perms.every(perm => permissions.value.includes(perm))
  }

  function hasRole(role: string): boolean {
    return roles.value.includes(role)
  }

  // 登录
  async function login(credentials: LoginCredentials) {
    const { res, error } = await useApi(authApi.login, credentials)

    if (res) {
      token.value = res.token
      userInfo.value = res.userInfo
      permissions.value = res.permissions
      roles.value = res.roles

      // 保存到本地存储
      localStorage.setItem('token', res.token)

      return res
    }

    if (error) {
      throw new Error(error.message)
    }
  }

  // 登出
  async function logout() {
    try {
      await useApi(authApi.logout)
    }
    finally {
      // 清除状态
      token.value = ''
      userInfo.value = null
      permissions.value = []
      roles.value = []

      // 清除本地存储
      localStorage.removeItem('token')

      // 跳转到登录页
      router.push('/login')
    }
  }

  // 获取用户信息
  async function getUserInfo() {
    if (!token.value) {
      return null
    }

    const { res, error } = await useApi(authApi.getUserInfo)

    if (res) {
      userInfo.value = res.userInfo
      permissions.value = res.permissions
      roles.value = res.roles
      return res
    }

    if (error) {
      // 如果获取用户信息失败，可能是 token 过期
      await logout()
      throw new Error(error.message)
    }
  }

  // 刷新权限
  async function refreshPermissions() {
    const { res, error } = await useApi(authApi.getPermissions)

    if (res) {
      permissions.value = res.permissions
      roles.value = res.roles
    }

    if (error) {
      console.error('刷新权限失败:', error.message)
    }
  }

  return {
    // 状态
    token: readonly(token),
    userInfo: readonly(userInfo),
    permissions: readonly(permissions),
    roles: readonly(roles),
    isAuthenticated,

    // 方法
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
    hasRole,
    login,
    logout,
    getUserInfo,
    refreshPermissions,
  }
})
```

## 权限指令

### 自定义权限指令

```typescript
// directives/permission.ts
import type { Directive } from 'vue'
import { useAuth } from '@/composables/useAuth'

// 权限指令
export const vPermission: Directive = {
  mounted(el, binding) {
    const { hasPermission } = useAuth()
    const permission = binding.value

    if (!hasPermission(permission)) {
      el.style.display = 'none'
    }
  },

  updated(el, binding) {
    const { hasPermission } = useAuth()
    const permission = binding.value

    if (!hasPermission(permission)) {
      el.style.display = 'none'
    }
    else {
      el.style.display = ''
    }
  },
}

// 使用示例
// <el-button v-permission="'user:create'">新增</el-button>
```

## 最佳实践

### 1. 权限粒度设计
- **功能权限**：控制功能模块的访问
- **操作权限**：控制具体操作的执行
- **数据权限**：控制数据的可见性和操作性
- **字段权限**：控制字段的显示和编辑

### 2. 权限检查原则
- **前端权限**：用于 UI 显示控制，提升用户体验
- **后端权限**：用于数据安全控制，确保系统安全
- **双重验证**：前后端都要进行权限验证

### 3. 性能优化
- **权限缓存**：避免重复的权限检查请求
- **懒加载**：按需加载权限相关的组件和数据
- **批量检查**：一次性检查多个权限，减少计算开销
