# OfferLink 项目结构与架构

## 项目概述

| 属性 | 值 |
|------|-----|
| 项目名 | OfferLink / getOffer |
| 版本 | 1.5.7 |
| 类型 | Chrome Extension (Manifest V3) |
| 仓库 | `syntaxsage2/getOffer` |
| 用途 | 自动填写招聘网站简历表单 |

## 目录结构

```
getOffer/
├── original/          # 原始扩展文件
├── extracted/         # 逆向拆分的 Webpack 模块（~140个）
│   └── panel/
│       ├── app_components.js  # 主 UI 组件（含原始 Po 组件）
│       ├── module_6012.js     # CSS 样式（含 .resume-preview-scroll）
│       ├── module_7965.js     # 剪贴板操作
│       ├── module_8054.js     # Ant Design copyable 相关
│       └── INDEX.txt          # 模块索引
├── src/               # 重建的源代码
│   ├── panel/         # 侧面板 UI（React + Ant Design）
│   ├── editor/        # 本地简历编辑器
│   ├── content/       # Content Script（表单填写 + panel 管理）
│   ├── background/    # Service Worker
│   ├── shared/        # 共享模块（resumeModel, resumeStorage）
│   └── popup/         # 弹窗
├── dist/              # 构建输出
├── webpack.config.js  # Webpack 配置
└── CROSS_REFERENCE.md # 原始↔重建代码映射
```

## 构建系统

- **Webpack 5** + **Babel**（target: Chrome 90）
- `splitChunks: false`（每个 entry point 自包含）
- `copy-webpack-plugin` 复制静态资源到 dist
- Entry points: `panel`, `content`, `background`, `popup`, `editor`

## manifest.json 关键配置

```json
{
  "manifest_version": 3,
  "permissions": ["storage", "activeTab", "tabs"],
  "content_scripts": [{ "matches": ["<all_urls>"], "js": ["content.js"] }],
  "side_panel": { "default_path": "panel.html" }
}
```

> **注意：** 没有 `clipboardWrite` 权限，剪贴板写入依赖 iframe 的 `allow="clipboard-write"` 属性。

## 逆向工程方法论

1. **js-beautify** 格式化原始 minified JS
2. 按 Webpack module boundary 拆分为独立文件（`extracted/`）
3. 通过搜索中文字符串定位功能代码
4. 分析变量命名模式（如 `Po=function`、`M=async function`）
5. 重建为可读的 React 组件（`src/`）

### 快速定位技巧

- 搜索中文关键词（如 `"已复制到剪贴板"`, `"基本信息"`）
- 搜索 CSS 类名（如 `resume-preview-scroll`）
- 搜索 API 调用（如 `navigator.clipboard`, `postMessage`）
- 搜索组件名模式（如 `Po=function`, `,Po=`）
