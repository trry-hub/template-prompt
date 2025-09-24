---
type: "always_apply"
---

# 项目配置和技术栈

## 技术栈配置

### 核心框架
- **Vue**: 3.5.18 (Composition API + `<script setup>`)
- **TypeScript**: 5.9 (严格模式)
- **Vite**: 7.1.2 (构建工具)

### UI 框架
- **Element Plus**: 2.10.7 (主 UI 库)
- **UnoCSS**: 66.4 (原子化 CSS)
- **@qxs-bns/components**: 自定义组件库

### 状态管理
- **Pinia**: 3.0.3 (状态管理)
- **VueUse**: 12.2 (组合式工具库)

### 路由和请求
- **Vue Router**: 4.5.0 (路由管理)
- **Axios**: 1.7.9 (HTTP 请求)

### 开发工具
- **ESLint**: @antfu/eslint-config (代码规范)
- **Stylelint**: 样式规范
- **Mock.js**: 数据模拟

## 项目结构

```
src/
├── api/                    # API 接口层
│   ├── index.ts           # API 入口和 useApi
│   ├── global-types.d.ts  # 全局类型定义
│   ├── modules/           # 接口模块
│   └── types/             # 类型定义
├── assets/                # 静态资源
│   ├── images/           # 图片资源
│   └── styles/           # 样式文件
├── components/            # 全局组件
├── composables/           # 组合函数 (已弃用，使用 utils/composables)
├── layouts/               # 布局组件
├── mock/                  # Mock 数据
│   ├── index.ts          # Mock 入口
│   └── modules/          # Mock 模块
├── router/                # 路由配置
│   ├── index.ts          # 路由入口
│   ├── guards.ts         # 路由守卫
│   └── modules/          # 路由模块
├── store/                 # 状态管理
│   ├── index.ts          # Store 入口
│   └── modules/          # Store 模块
├── utils/                 # 工具函数
│   ├── composables/      # 组合函数
│   ├── dayjs.ts          # 日期工具
│   ├── directive.ts      # 自定义指令
│   ├── eventBus.ts       # 事件总线
│   ├── qxs-bns.ts        # QXS 组件库配置
│   └── watermark.ts      # 水印工具
├── views/                 # 页面组件
├── App.vue               # 根组件
└── main.ts               # 应用入口
```

## 全局自动引入

### Vue APIs
```typescript
// 自动引入的 Vue APIs
ref, reactive, computed, watch, watchEffect, onMounted, onUnmounted,
nextTick, defineProps, defineEmits, defineOptions, etc.
```

### 组合函数
```typescript
// 自动引入的 Composables
useRouter, useRoute, useAuth, usePagination, useMessage, useWatermark, etc.
```

### 组件
```typescript
// 自动引入的组件
Auth, AuthAll, ImagePreview, NotAllowed, PageHeader, PageMain
```

### 工具库
```typescript
// 自动引入的工具
ElMessage, ElMessageBox, ElNotification, ElLoading, dayjs, etc.
```

## 环境配置

### 开发环境
- 热重载和快速刷新
- Mock 数据支持
- 开发工具集成
- 错误边界和调试

### 构建配置
- TypeScript 严格检查
- 代码分割和懒加载
- 资源优化和压缩
- 环境变量管理

## 代码规范配置

### ESLint 规则
- @antfu/eslint-config
- TypeScript 严格模式
- Vue 3 Composition API 规范
- 自动格式化和修复

### Stylelint 规则
- SCSS 语法支持
- BEM 命名规范
- CSS 变量使用
- 响应式设计规范

### Git 规范
- Conventional Commits
- Pre-commit 钩子
- 代码质量检查
- 自动化测试

## 浏览器支持

### 目标浏览器
- Chrome >= 87
- Firefox >= 78
- Safari >= 14
- Edge >= 88

### 兼容性处理
- Polyfill 自动注入
- CSS 前缀自动添加
- 现代 JS 特性支持
- 响应式设计适配
