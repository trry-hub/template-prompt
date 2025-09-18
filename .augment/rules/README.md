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
├── USAGE.md                     # 📚 使用指南
├── prompt-writing-standards.md  # 📝 提示词编写标准
│
├── auto/                        # 🤖 自动规则（根据文件类型自动应用）
│   ├── vue-components.md        # Vue组件开发规则
│   └── typescript-standards.md  # TypeScript开发标准
│
├── always/                      # 📌 始终应用的规则
│   └── code-standards.md        # 代码规范和命名约定
│
└── manual/                      # 👤 手动应用的规则（通过@mention引用）
    ├── business/                # 💼 业务逻辑规则
    │   └── views.md             # 页面业务逻辑规则
    └── shared/                  # 🔄 可共享的规则
        ├── project-config.md    # 项目配置和技术栈
        ├── api-patterns.md      # API使用规范
        ├── pagination-patterns.md # 分页使用规范
        ├── error-handling.md    # 错误处理规范
        └── auth-patterns.md     # 权限控制规范
```

## 🔍 规则类型说明

Augment支持三种类型的规则：

1. **Always规则**：内容将包含在每个用户消息中，无需手动引用
2. **Manual规则**：需要通过@mention手动引用规则文件
3. **Auto规则**：Agent将根据描述字段自动检测并附加规则

要在文件开头注明
---
type: "agent_requested" | "manual" | "always_apply"
description: "描述"
---
