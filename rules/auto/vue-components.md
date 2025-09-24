---
type: "agent_requested"
description: "Vue组件开发规则"
---

# Vue组件开发规则

## 描述
本规则适用于Vue组件开发，当检测到.vue文件时自动应用。

## 组件开发规范
- **组件结构**：使用`<script setup lang="ts">`语法
- **类型安全**：所有props和响应式数据必须有TypeScript类型定义
- **命名规范**：组件使用PascalCase，文件使用kebab-case
- **样式隔离**：使用`scoped`或CSS模块避免样式污染
- **响应式API**：优先使用`ref`、`reactive`和`computed`

## 最佳实践
- **单一职责**：每个组件只负责一个功能
- **可复用性**：设计可在多个场景下复用的组件
- **可测试性**：组件逻辑应易于单元测试
- **错误处理**：提供完善的错误处理机制
- **加载状态**：实现加载状态和骨架屏
