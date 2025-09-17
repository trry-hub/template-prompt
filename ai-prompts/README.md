# AI Prompts 模块化系统

## 🏗️ 系统架构

这是一个完全模块化的 AI Prompt 系统，按照**业务逻辑**和**样式设计**分离的原则设计，支持精准的开发需求匹配。

### 📁 目录结构
```
docs/ai-prompts/
├── README.md                    # 📖 系统说明
├── main-assistant-new.md        # 🎯 主助手（推荐使用）
│
├── shared/                      # 🔄 公共规范（所有模块共享）
│   ├── project-config.md        # 项目配置和技术栈
│   ├── code-standards.md        # 代码规范和命名约定
│   ├── api-patterns.md          # useApi 使用规范
│   ├── pagination-patterns.md   # usePagination 使用规范
│   ├── error-handling.md        # 错误处理规范
│   └── auth-patterns.md         # 权限控制规范
│
├── business/                    # 💼 业务逻辑专家（不涉及样式）
│   ├── components.md            # 组件业务逻辑
│   ├── views.md                 # 页面业务逻辑
│   ├── composables.md           # 组合函数
│   ├── api.md                   # API 接口
│   ├── types.md                 # 类型定义
│   ├── store.md                 # 状态管理
│   ├── router.md                # 路由配置
│   └── mock.md                  # Mock 数据
│
├── styling/                     # 🎨 样式专家（不涉及业务）
│   ├── components.md            # 组件样式
│   ├── views.md                 # 页面样式
│   ├── layouts.md               # 布局样式
│   ├── global.md                # 全局样式
│   ├── themes.md                # 主题定制
│   └── responsive.md            # 响应式设计
│
└── templates/                   # 📋 完整模板（业务+样式）
    ├── crud-page.md             # CRUD 页面完整模板
    ├── form-component.md        # 表单组件完整模板
    ├── list-component.md        # 列表组件完整模板
    └── dashboard.md             # 仪表盘完整模板
```

## 🎯 核心特性

### ✅ 业务与样式分离
- **业务修改**：只修改 `business/` 模块，完全不动样式
- **样式修改**：只修改 `styling/` 模块，完全不动业务逻辑
- **完整开发**：使用 `templates/` 模块或主助手

### ✅ 公共规范抽离
- **useApi 规范**：统一的 API 调用模式，所有模块共享
- **usePagination 规范**：统一的分页处理模式
- **代码规范**：命名约定、组件结构、TypeScript 规范
- **项目配置**：技术栈、目录结构、环境配置

### ✅ 精准模块匹配
- **目录对应**：每个项目目录都有对应的专业模块
- **功能对应**：每种开发需求都有专门的专家模块
- **智能路由**：主助手自动选择合适的专业模块

## 🚀 使用方式

### 1. 主助手（推荐 90% 场景）
使用 `main-assistant.md` - 包含所有项目规范的全能助手：

```
👤 用户：我要创建一个用户管理页面，包含列表、搜索、新增、编辑、删除功能
🤖 AI：[自动生成页面组件 + Composable + API 接口 + 类型定义的完整代码]

👤 用户：帮我写一个文件上传组件，支持拖拽和进度显示
🤖 AI：[生成完整的上传组件，包含所有必要功能]

👤 用户：我需要一个订单状态管理的 Composable
🤖 AI：[生成符合项目规范的 Composable 代码]
```

### 2. 业务逻辑专家模式
```
使用：business/*.md

场景：
👤 "优化用户列表的搜索逻辑，添加高级筛选功能"
🤖 使用：business/views.md + shared/api-patterns.md
🤖 输出：只有业务逻辑代码，不包含任何样式
```

### 3. 样式设计专家模式
```
使用：styling/*.md

场景：
👤 "调整表格组件的样式，改为卡片布局，适配移动端"
🤖 使用：styling/components.md + shared/code-standards.md
🤖 输出：只有样式代码，不包含任何业务逻辑
```

### 4. 完整模板模式
```
使用：templates/*.md

场景：
👤 "我需要一个标准的 CRUD 页面"
🤖 使用：templates/crud-page.md
🤖 输出：包含业务逻辑 + 样式的完整实现
```

## 📋 开发规范

### 🔄 公共规范（所有模块遵循）

#### API 调用规范
```typescript
// ✅ 正确使用 useApi
const { res, error } = await useApi(moduleApi.list, params)
if (res) {
  // 处理成功响应
  dataList.value = res.list
  pagination.value.total = res.total
}
if (error) {
  // 处理错误响应
  ElMessage.error(error.message || '获取数据失败')
}
```

#### 分页管理规范
```typescript
// ✅ 正确使用 usePagination
const { pagination, getParams, onSizeChange, onCurrentChange, onSortChange } = usePagination()

// 获取数据时使用 getParams()
const params = {
  ...getParams(), // 包含 { page, size, sortBy, sortOrder }
  ...searchParams.value, // 搜索参数
}

// 分页变化时使用对应方法
function handleCurrentChange(page: number) {
  onCurrentChange(page).then(() => getDataList())
}
```

#### 代码规范
- **组件语法**：统一使用 `<script setup lang="ts">` 语法
- **命名规范**：组件 PascalCase，文件 kebab-case，变量 camelCase
- **类型安全**：所有数据和函数必须有完整的 TypeScript 类型定义
- **样式规范**：使用 `qxs-` 前缀 + BEM 命名，优先使用 CSS 变量

## 🔧 技术栈配置

- **Vue**: 3.5.18 (Composition API + `<script setup>`)
- **TypeScript**: 5.9 (严格模式)
- **Element Plus**: 2.10.7 (主 UI 库)
- **UnoCSS**: 66.4 (原子化 CSS)
- **@qxs-bns**: 自定义组件库
- **Pinia**: 3.0.3 (状态管理)
- **Vue Router**: 4.5.0 (路由管理)
- **Axios**: 1.7.9 (HTTP 请求)

## 🎉 系统优势

### 1. **模块化设计**
- 功能清晰分离，易于维护和扩展
- 公共规范统一管理，避免重复
- 专业模块精准匹配，提高开发效率

### 2. **业务样式分离**
- 业务开发不影响样式
- 样式调整不影响业务逻辑
- 支持独立的业务重构和视觉改版

### 3. **智能路由**
- 主助手自动分析需求
- 智能选择合适的专业模块
- 生成符合项目规范的高质量代码

### 4. **完全定制**
- 基于项目实际技术栈和架构
- 包含真实的组件示例和代码模板
- 遵循项目的编码规范和最佳实践

## 🚀 快速开始

### 推荐使用方式
1. **主助手开发**：使用 `main-assistant.md` 处理 90% 的开发场景
2. **业务优化**：使用 `business/*.md` 专家进行业务逻辑开发
3. **样式调整**：使用 `styling/*.md` 专家进行样式设计
4. **完整功能**：使用 `templates/*.md` 快速生成标准功能模块

### 使用建议
- **新功能开发**：优先使用主助手或完整模板
- **业务优化**：使用对应的业务逻辑专家
- **样式改版**：使用对应的样式设计专家
- **规范学习**：参考 `shared/` 目录下的公共规范

现在你可以根据具体需求选择合适的模块开始开发了！

---

## 📝 Prompt 编写规范

为了保持 prompt 质量和一致性，请遵循 [prompt-writing-standards.md](prompt-writing-standards.md) 中的编写规范：

### 核心原则
1. **结构标准化**：使用统一的章节结构（Role、Skills、Rules、Templates）
2. **内容实用性**：提供具体可用的代码模板和最佳实践
3. **规范一致性**：遵循项目的技术栈和编码规范
4. **持续更新**：定期更新技术栈版本和最佳实践

### 质量标准
- ✅ 角色定义清晰明确
- ✅ 技能列表完整实用
- ✅ 规则约束具体可执行
- ✅ 代码模板完整可用
- ✅ 最佳实践覆盖常见场景
