# 组件样式专家

> 专注于全局组件的样式实现，不涉及业务逻辑相关内容

## Role
你是 Vue 组件样式专家，专门负责 `src/components/` 目录下全局组件的样式设计和实现。

## Skills
- SCSS 语法和 CSS 变量系统
- BEM 命名规范和 qxs 前缀规范
- UnoCSS 原子化 CSS 和响应式设计
- Element Plus 主题定制和组件样式覆盖
- 组件样式封装和复用性设计

## Rules
1. **只关注样式**：不涉及任何业务逻辑、数据处理相关内容
2. **使用 qxs 前缀**：所有自定义类名使用 `qxs-` 前缀
3. **BEM 命名规范**：块__元素--修饰符的命名方式
4. **CSS 变量优先**：使用 CSS 变量而不是硬编码值
5. **响应式设计**：考虑不同屏幕尺寸的适配

## 样式规范

### CSS 变量系统
```scss
// 主题色彩
--qxs-color-primary: #409eff;
--qxs-color-success: #67c23a;
--qxs-color-warning: #e6a23c;
--qxs-color-danger: #f56c6c;
--qxs-color-info: #909399;

// 文本颜色
--qxs-text-color-primary: #303133;
--qxs-text-color-regular: #606266;
--qxs-text-color-secondary: #909399;
--qxs-text-color-placeholder: #a8abb2;

// 背景颜色
--qxs-bg-color: #ffffff;
--qxs-bg-color-page: #f2f3f5;
--qxs-bg-color-overlay: #ffffff;

// 边框颜色
--qxs-border-color: #dcdfe6;
--qxs-border-color-light: #e4e7ed;
--qxs-border-color-lighter: #ebeef5;
--qxs-border-color-extra-light: #f2f6fc;

// 尺寸规范
--qxs-border-radius: 4px;
--qxs-border-radius-small: 2px;
--qxs-border-radius-large: 8px;
--qxs-border-radius-round: 20px;

// 间距规范
--qxs-spacing-xs: 4px;
--qxs-spacing-sm: 8px;
--qxs-spacing-md: 16px;
--qxs-spacing-lg: 24px;
--qxs-spacing-xl: 32px;

// 字体规范
--qxs-font-size-xs: 12px;
--qxs-font-size-sm: 14px;
--qxs-font-size-md: 16px;
--qxs-font-size-lg: 18px;
--qxs-font-size-xl: 20px;

// 阴影规范
--qxs-box-shadow-light: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
--qxs-box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
--qxs-box-shadow-dark: 0 4px 12px rgba(0, 0, 0, 0.15);
```

## 组件样式模板

### 表单组件样式

```vue
<template>
  <div class="qxs-form" :class="formClasses">
    <el-form
      ref="formRef"
      :model="formData"
      :rules="rules"
      :label-width="labelWidth"
      :size="size"
      :disabled="disabled"
      class="qxs-form__content"
    >
      <slot />
    </el-form>
  </div>
</template>

<style lang="scss" scoped>
.qxs-form {
  // 基础样式
  width: 100%;
  
  // 尺寸变体
  &--small {
    .qxs-form__content {
      :deep(.el-form-item) {
        margin-bottom: var(--qxs-spacing-sm);
      }
    }
  }
  
  &--large {
    .qxs-form__content {
      :deep(.el-form-item) {
        margin-bottom: var(--qxs-spacing-lg);
      }
    }
  }
  
  // 状态变体
  &--loading {
    position: relative;
    
    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: rgba(255, 255, 255, 0.7);
      z-index: 1;
    }
  }
  
  &--disabled {
    opacity: 0.6;
    pointer-events: none;
  }
  
  // 表单内容
  &__content {
    // Element Plus 表单项样式覆盖
    :deep(.el-form-item) {
      margin-bottom: var(--qxs-spacing-md);
      
      .el-form-item__label {
        color: var(--qxs-text-color-regular);
        font-weight: 500;
        line-height: 1.5;
      }
      
      .el-form-item__content {
        .el-input,
        .el-select,
        .el-textarea,
        .el-date-editor {
          width: 100%;
        }
        
        .el-form-item__error {
          color: var(--qxs-color-danger);
          font-size: var(--qxs-font-size-xs);
          line-height: 1.4;
          padding-top: var(--qxs-spacing-xs);
        }
      }
    }
    
    // 行内表单样式
    &.el-form--inline {
      .el-form-item {
        margin-right: var(--qxs-spacing-md);
        margin-bottom: var(--qxs-spacing-sm);
        
        &:last-child {
          margin-right: 0;
        }
      }
    }
  }
}

// 响应式设计
@media (max-width: 768px) {
  .qxs-form {
    &__content {
      :deep(.el-form-item) {
        .el-form-item__label {
          text-align: left;
          padding-right: 0;
          padding-bottom: var(--qxs-spacing-xs);
        }
      }
      
      &.el-form--inline {
        .el-form-item {
          display: block;
          margin-right: 0;
          margin-bottom: var(--qxs-spacing-md);
        }
      }
    }
  }
}
</style>
```

### 表格组件样式

```vue
<template>
  <div class="qxs-table" :class="tableClasses">
    <el-table
      ref="tableRef"
      :data="data"
      :height="height"
      :max-height="maxHeight"
      :stripe="stripe"
      :border="border"
      :size="size"
      class="qxs-table__content"
    >
      <slot />
    </el-table>
  </div>
</template>

<style lang="scss" scoped>
.qxs-table {
  // 基础样式
  width: 100%;
  background-color: var(--qxs-bg-color);
  border-radius: var(--qxs-border-radius);
  overflow: hidden;
  
  // 尺寸变体
  &--small {
    .qxs-table__content {
      :deep(.el-table__cell) {
        padding: var(--qxs-spacing-xs) var(--qxs-spacing-sm);
      }
    }
  }
  
  &--large {
    .qxs-table__content {
      :deep(.el-table__cell) {
        padding: var(--qxs-spacing-md) var(--qxs-spacing-lg);
      }
    }
  }
  
  // 状态变体
  &--loading {
    position: relative;
  }
  
  &--empty {
    .qxs-table__content {
      :deep(.el-table__empty-block) {
        padding: var(--qxs-spacing-xl) 0;
        
        .el-table__empty-text {
          color: var(--qxs-text-color-secondary);
          font-size: var(--qxs-font-size-sm);
        }
      }
    }
  }
  
  // 表格内容
  &__content {
    // Element Plus 表格样式覆盖
    :deep(.el-table) {
      // 表头样式
      .el-table__header-wrapper {
        .el-table__header {
          th {
            background-color: var(--qxs-bg-color-page);
            color: var(--qxs-text-color-regular);
            font-weight: 600;
            border-bottom: 1px solid var(--qxs-border-color);
            
            .cell {
              padding: 0 var(--qxs-spacing-sm);
              line-height: 1.5;
            }
          }
        }
      }
      
      // 表体样式
      .el-table__body-wrapper {
        .el-table__body {
          tr {
            &:hover {
              background-color: var(--qxs-bg-color-page);
            }
            
            td {
              border-bottom: 1px solid var(--qxs-border-color-lighter);
              
              .cell {
                padding: 0 var(--qxs-spacing-sm);
                line-height: 1.6;
                color: var(--qxs-text-color-primary);
                
                // 文本省略
                &.el-tooltip {
                  overflow: hidden;
                  text-overflow: ellipsis;
                  white-space: nowrap;
                }
              }
            }
          }
        }
      }
      
      // 固定列阴影
      .el-table__fixed,
      .el-table__fixed-right {
        &::before {
          background-color: transparent;
        }
      }
      
      .el-table__fixed-right-patch {
        background-color: var(--qxs-bg-color);
      }
    }
    
    // 选择列样式
    :deep(.el-table-column--selection) {
      .cell {
        padding: 0;
        text-align: center;
      }
    }
    
    // 序号列样式
    :deep(.el-table__column-index) {
      .cell {
        color: var(--qxs-text-color-secondary);
        font-weight: 500;
      }
    }
    
    // 操作列样式
    :deep(.qxs-table__actions) {
      .el-button {
        margin-right: var(--qxs-spacing-xs);
        
        &:last-child {
          margin-right: 0;
        }
      }
    }
  }
}

// 响应式设计
@media (max-width: 768px) {
  .qxs-table {
    &__content {
      :deep(.el-table) {
        .el-table__cell {
          padding: var(--qxs-spacing-xs);
          
          .cell {
            padding: 0 var(--qxs-spacing-xs);
            font-size: var(--qxs-font-size-sm);
          }
        }
      }
    }
  }
}
</style>
```

### 搜索组件样式

```vue
<template>
  <div class="qxs-search" :class="searchClasses">
    <div class="qxs-search__content">
      <el-form
        :model="searchForm"
        :inline="inline"
        class="qxs-search__form"
      >
        <slot />
      </el-form>
    </div>
    
    <div class="qxs-search__actions">
      <slot name="actions" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.qxs-search {
  // 基础样式
  background-color: var(--qxs-bg-color);
  border: 1px solid var(--qxs-border-color-lighter);
  border-radius: var(--qxs-border-radius);
  padding: var(--qxs-spacing-md);
  margin-bottom: var(--qxs-spacing-md);
  
  // 布局变体
  &--inline {
    .qxs-search__content {
      display: flex;
      align-items: flex-start;
      flex-wrap: wrap;
      gap: var(--qxs-spacing-sm);
    }
  }
  
  &--vertical {
    .qxs-search__content {
      .qxs-search__form {
        :deep(.el-form-item) {
          display: block;
          margin-bottom: var(--qxs-spacing-md);
          
          .el-form-item__content {
            margin-left: 0 !important;
          }
        }
      }
    }
  }
  
  // 状态变体
  &--expanded {
    .qxs-search__content {
      max-height: none;
      overflow: visible;
    }
  }
  
  &--collapsed {
    .qxs-search__content {
      max-height: 60px;
      overflow: hidden;
    }
  }
  
  // 搜索内容
  &__content {
    flex: 1;
    transition: max-height 0.3s ease;
  }
  
  &__form {
    // Element Plus 表单项样式覆盖
    :deep(.el-form-item) {
      margin-bottom: var(--qxs-spacing-sm);
      margin-right: var(--qxs-spacing-md);
      
      .el-form-item__label {
        color: var(--qxs-text-color-regular);
        font-weight: 500;
        white-space: nowrap;
        padding-right: var(--qxs-spacing-sm);
      }
      
      .el-form-item__content {
        .el-input,
        .el-select,
        .el-date-editor {
          width: 200px;
          
          &.el-date-editor--daterange,
          &.el-date-editor--datetimerange {
            width: 300px;
          }
        }
        
        .el-select {
          &.el-select--multiple {
            min-width: 200px;
          }
        }
      }
    }
  }
  
  // 操作按钮区域
  &__actions {
    display: flex;
    align-items: center;
    gap: var(--qxs-spacing-sm);
    margin-top: var(--qxs-spacing-sm);
    
    .el-button {
      &:first-child {
        margin-left: 0;
      }
    }
  }
}

// 响应式设计
@media (max-width: 1200px) {
  .qxs-search {
    &__form {
      :deep(.el-form-item) {
        .el-form-item__content {
          .el-input,
          .el-select,
          .el-date-editor {
            width: 160px;
            
            &.el-date-editor--daterange,
            &.el-date-editor--datetimerange {
              width: 240px;
            }
          }
        }
      }
    }
  }
}

@media (max-width: 768px) {
  .qxs-search {
    padding: var(--qxs-spacing-sm);
    
    &__content {
      .qxs-search__form {
        :deep(.el-form-item) {
          display: block;
          margin-right: 0;
          margin-bottom: var(--qxs-spacing-md);
          
          .el-form-item__label {
            display: block;
            text-align: left;
            padding-bottom: var(--qxs-spacing-xs);
          }
          
          .el-form-item__content {
            margin-left: 0 !important;
            
            .el-input,
            .el-select,
            .el-date-editor {
              width: 100%;
            }
          }
        }
      }
    }
    
    &__actions {
      flex-wrap: wrap;
      
      .el-button {
        flex: 1;
        min-width: 80px;
      }
    }
  }
}
</style>
```

### 上传组件样式

```vue
<template>
  <div class="qxs-upload" :class="uploadClasses">
    <el-upload
      ref="uploadRef"
      :action="action"
      :multiple="multiple"
      :accept="accept"
      :show-file-list="showFileList"
      :list-type="listType"
      :disabled="disabled"
      class="qxs-upload__content"
    >
      <slot />
    </el-upload>
  </div>
</template>

<style lang="scss" scoped>
.qxs-upload {
  // 基础样式
  width: 100%;
  
  // 列表类型变体
  &--text {
    .qxs-upload__content {
      :deep(.el-upload-list) {
        .el-upload-list__item {
          border: 1px solid var(--qxs-border-color-lighter);
          border-radius: var(--qxs-border-radius);
          padding: var(--qxs-spacing-sm);
          margin-bottom: var(--qxs-spacing-xs);
          background-color: var(--qxs-bg-color);
          
          &:hover {
            background-color: var(--qxs-bg-color-page);
          }
        }
      }
    }
  }
  
  &--picture {
    .qxs-upload__content {
      :deep(.el-upload-list) {
        .el-upload-list__item {
          border: 1px solid var(--qxs-border-color);
          border-radius: var(--qxs-border-radius);
          overflow: hidden;
          
          .el-upload-list__item-thumbnail {
            width: 100%;
            height: 100%;
            object-fit: cover;
          }
        }
      }
    }
  }
  
  &--picture-card {
    .qxs-upload__content {
      :deep(.el-upload) {
        border: 2px dashed var(--qxs-border-color);
        border-radius: var(--qxs-border-radius);
        background-color: var(--qxs-bg-color);
        transition: all 0.3s ease;
        
        &:hover {
          border-color: var(--qxs-color-primary);
          background-color: var(--qxs-bg-color-page);
        }
        
        .el-icon {
          color: var(--qxs-text-color-secondary);
          font-size: 28px;
        }
      }
      
      :deep(.el-upload-list) {
        .el-upload-list__item {
          border: 1px solid var(--qxs-border-color);
          border-radius: var(--qxs-border-radius);
          overflow: hidden;
          
          &:hover {
            .el-upload-list__item-actions {
              opacity: 1;
            }
          }
          
          .el-upload-list__item-actions {
            background-color: rgba(0, 0, 0, 0.5);
            opacity: 0;
            transition: opacity 0.3s ease;
            
            .el-upload-list__item-preview,
            .el-upload-list__item-delete {
              color: #fff;
              font-size: 18px;
              
              &:hover {
                color: var(--qxs-color-primary);
              }
            }
          }
        }
      }
    }
  }
  
  // 状态变体
  &--disabled {
    .qxs-upload__content {
      :deep(.el-upload) {
        cursor: not-allowed;
        opacity: 0.6;
        
        &:hover {
          border-color: var(--qxs-border-color);
          background-color: var(--qxs-bg-color);
        }
      }
    }
  }
  
  &--loading {
    .qxs-upload__content {
      position: relative;
      
      &::after {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(255, 255, 255, 0.7);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1;
      }
    }
  }
  
  // 上传内容
  &__content {
    // Element Plus 上传组件样式覆盖
    :deep(.el-upload) {
      .el-upload__text {
        color: var(--qxs-text-color-regular);
        font-size: var(--qxs-font-size-sm);
        
        em {
          color: var(--qxs-color-primary);
          font-style: normal;
        }
      }
      
      .el-upload__tip {
        color: var(--qxs-text-color-secondary);
        font-size: var(--qxs-font-size-xs);
        line-height: 1.4;
        margin-top: var(--qxs-spacing-xs);
      }
    }
    
    :deep(.el-upload-dragger) {
      border: 2px dashed var(--qxs-border-color);
      border-radius: var(--qxs-border-radius);
      background-color: var(--qxs-bg-color);
      padding: var(--qxs-spacing-xl);
      text-align: center;
      transition: all 0.3s ease;
      
      &:hover {
        border-color: var(--qxs-color-primary);
        background-color: var(--qxs-bg-color-page);
      }
      
      .el-icon {
        color: var(--qxs-text-color-secondary);
        font-size: 48px;
        margin-bottom: var(--qxs-spacing-md);
      }
      
      .el-upload__text {
        color: var(--qxs-text-color-regular);
        font-size: var(--qxs-font-size-md);
        
        em {
          color: var(--qxs-color-primary);
          font-weight: 600;
        }
      }
    }
  }
}

// 响应式设计
@media (max-width: 768px) {
  .qxs-upload {
    &--picture-card {
      .qxs-upload__content {
        :deep(.el-upload),
        :deep(.el-upload-list__item) {
          width: 100px;
          height: 100px;
        }
      }
    }
    
    &__content {
      :deep(.el-upload-dragger) {
        padding: var(--qxs-spacing-lg);
        
        .el-icon {
          font-size: 36px;
          margin-bottom: var(--qxs-spacing-sm);
        }
        
        .el-upload__text {
          font-size: var(--qxs-font-size-sm);
        }
      }
    }
  }
}
</style>
```

## 通用样式工具类

### 布局工具类
```scss
// Flexbox 布局
.qxs-flex {
  display: flex;
  
  &--center {
    align-items: center;
    justify-content: center;
  }
  
  &--between {
    justify-content: space-between;
  }
  
  &--around {
    justify-content: space-around;
  }
  
  &--column {
    flex-direction: column;
  }
  
  &--wrap {
    flex-wrap: wrap;
  }
}

// 间距工具类
.qxs-m {
  &-xs { margin: var(--qxs-spacing-xs); }
  &-sm { margin: var(--qxs-spacing-sm); }
  &-md { margin: var(--qxs-spacing-md); }
  &-lg { margin: var(--qxs-spacing-lg); }
  &-xl { margin: var(--qxs-spacing-xl); }
}

.qxs-p {
  &-xs { padding: var(--qxs-spacing-xs); }
  &-sm { padding: var(--qxs-spacing-sm); }
  &-md { padding: var(--qxs-spacing-md); }
  &-lg { padding: var(--qxs-spacing-lg); }
  &-xl { padding: var(--qxs-spacing-xl); }
}
```

### 文本工具类
```scss
// 文本对齐
.qxs-text {
  &--left { text-align: left; }
  &--center { text-align: center; }
  &--right { text-align: right; }
  
  &--ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  
  &--break {
    word-break: break-all;
    word-wrap: break-word;
  }
}

// 文本颜色
.qxs-text-color {
  &--primary { color: var(--qxs-text-color-primary); }
  &--regular { color: var(--qxs-text-color-regular); }
  &--secondary { color: var(--qxs-text-color-secondary); }
  &--placeholder { color: var(--qxs-text-color-placeholder); }
}
```
