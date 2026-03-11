# 简历预览组件增强 — Walkthrough

## 变更概览

从原始逆向代码中的 `Po` 组件还原了完整的简历预览功能，并与编辑器页面的数据结构对齐。

## 文件变更

| 文件 | 类型 | 行数 | 说明 |
|------|------|------|------|
| [useCopyable.js](file:///Users/zane/Downloads/inbox/getOffer/src/panel/useCopyable.js) | 新建 | 67 | 点击复制 Hook（clipboard API + textarea fallback） |
| [ResumePreview.jsx](file:///Users/zane/Downloads/inbox/getOffer/src/panel/ResumePreview.jsx) | 新建 | ~485 | 完整预览组件（13 section + 导航 + Avatar） |
| [panel.css](file:///Users/zane/Downloads/inbox/getOffer/src/panel/panel.css) | 修改 | +86 | 导航面板 + section 标题样式 |
| [App.jsx](file:///Users/zane/Downloads/inbox/getOffer/src/panel/App.jsx) | 修改 | -85 | 移除旧版简化 ResumePreview，集成新组件 |

## 核心功能

### 点击复制（`useCopyable` Hook）
- `copyToClipboard(text)` — 异步复制 + 成功/失败提示
- `copyableProps(value)` — 工厂函数绑定 `onClick` + 鼠标悬停高亮
- `COPYABLE_STYLE` — 游标指针 + 圆角 + 过渡动画

### 13 个数据 Section
基本信息 → 求职期望 → 教育背景 → 实习经历 → 项目经验 → 工作经历 → 在校职务 → 校园活动 → 家庭成员 → 获奖经历 → 语言能力 → 证书/论文/专利 → 自我评价

> 只显示有数据的 section，空数据自动隐藏。

### 快速导航
- "导航" 按钮切换浮动面板
- 点击导航项平滑滚动到对应 section
- 滚动时自动高亮当前 section
- "回到顶部" 快捷入口

## 验证结果

```
webpack 5.105.4 compiled with 3 warnings in 8957 ms
Exit code: 0
```

✅ 构建成功，0 错误。3 个 warning 均为 bundle size 提示（Ant Design 体积大，属预期行为）。
