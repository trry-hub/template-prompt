---
type: "manual"
---

# Augment 规则系统

## 🏗️ 系统架构

这是一个基于Augment的规则系统，按照官方支持的三种规则类型进行组织，支持精准的开发需求匹配和自动化应用。

### 📁 目录结构
```
.augment/rules/
├── README.md                    # 📖 系统说明
├── prompt-standards.md          # 📝 提示词编写标准
│
├── always/                      # 📌 始终应用的规则
│   ├── main-assistant.md        # 🎯 主开发助手规则
│   ├── code-standards.md        # 📋 代码规范和命名约定
│   └── project-config.md        # ⚙️ 项目配置和技术栈
│
└── auto/                        # 🤖 自动规则（根据描述自动触发）
    ├── code-writing-order.md    # 📝 完整编码规范（核心）
    ├── component-interaction.md # 🔄 组件交互逻辑规范
    ├── api-patterns.md          # 🌐 API使用规范
    ├── auth-patterns.md         # 🔐 权限控制规范
    ├── pagination-patterns.md   # 📄 分页使用规范
    ├── typescript-standards.md  # 🔧 TypeScript开发标准
    ├── vue-components.md        # 🎨 Vue组件开发规则
    └── component-optimization.md # ⚡ 组件优化规范
```

## 🚨 核心规则说明

### Always Rules (始终应用)
这些规则在每次对话中都会自动应用，是项目的基础规范：

1. **main-assistant.md** - 主开发助手的核心职责和规则
2. **code-standards.md** - 基础代码标准和命名约定
3. **project-config.md** - 项目技术栈和配置规范

### Auto Rules (自动触发)
根据用户请求内容自动触发的专业规则：

| 触发描述 | 规则文件 | 应用场景 |
|---------|---------|---------|
| "代码编写顺序" | code-writing-order.md | 🎯 Vue组件开发（完整编码规范） |
| "组件交互逻辑开发规范" | component-interaction.md | 🔄 交互功能开发 |
| "useApi 使用规范" | api-patterns.md | 🌐 API调用相关 |
| "auth pattern" | auth-patterns.md | 🔐 权限控制相关 |
| "usePagination 使用规范" | pagination-patterns.md | 📄 分页功能相关 |
| "TypeScript开发标准" | typescript-standards.md | 🔧 TypeScript开发 |
| "Vue组件开发规则" | vue-components.md | 🎨 Vue组件开发 |
| "组件规范" | component-optimization.md | ⚡ 组件优化相关 |

## 🔍 规则类型说明

Augment支持三种类型的规则：

1. **Always规则**：内容将包含在每个用户消息中，无需手动引用
2. **Manual规则**：需要通过@mention手动引用规则文件
3. **Auto规则**：Agent将根据描述字段自动检测并附加规则

## 📋 代码编写标准流程

### 🚨 严格的代码顺序 (必须遵循)

所有 Vue 组件必须按照以下顺序编写代码：

```typescript
<script setup lang="ts">
// 1. 类型导入
import type { EvaluateInfo } from './types'

// 2. 项目导入
import { evaluateI18n } from './constants'

// 3. 组件导入
import EvaluateSettings from './EvaluateSettings.vue'

// 4. 组件配置
defineOptions({
  name: 'BasicAssessment'
})

// 5. Emits (必须在 Props 之前)
const emit = defineEmits<{
  change: [data: any]
}>()

// 6. Props
const {
  maxItems = 2
} = defineProps<{
  maxItems?: number
}>()

// 7. Hooks/Composables
const router = useRouter()

// 8. 常量配置
const MAX_ITEMS = 2

// 9. 响应式数据
const items = ref<any[]>([])

// 10. 计算属性
const canAddMore = computed(() => items.value.length < MAX_ITEMS)

// 11. 方法定义
function addItem(): void {
  if (canAddMore.value) {
    items.value.push(null)
  }
}

// 12. 监听器
watch(items, (newItems) => {
  emit('change', newItems)
}, { deep: true })

// 13. 生命周期
onMounted(() => {
  console.log('Component mounted')
})
</script>
```

### 🔧 规则文件格式

要在文件开头注明：
```yaml
---
type: "agent_requested" | "manual" | "always_apply"
description: "描述"
---
```

### 🎯 使用建议

1. **开发新组件时**：严格按照 `code-writing-order.md` 完整编码规范编写
2. **添加交互功能时**：参考 `component-interaction.md` 规范
3. **API 调用时**：遵循 `api-patterns.md` 模式
4. **代码审查时**：使用 `code-writing-order.md` 中的检查清单

### 📋 核心特性

- **🚨 严格的15步代码编写顺序** - 确保代码结构一致性
- **✅ 完整的代码质量检查清单** - 涵盖类型安全、命名规范、错误处理等
- **🔧 最佳实践指南** - 包含样式规范、业务逻辑检查等
- **📝 实用代码模板** - 提供标准的组件开发模板

这个规则体系确保了代码的一致性、可维护性和高质量。
