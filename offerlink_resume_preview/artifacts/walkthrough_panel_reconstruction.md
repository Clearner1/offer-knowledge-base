# OfferLink Chrome Extension — 逆向重建项目总结

## 1. 项目概述

逆向重建 [OfferLink](https://github.com/syntaxsage2/getOffer) Chrome 扩展，一个自动填写招聘网站简历的工具。原始代码为 webpack 编译后的混淆 JS，已成功还原为可读的模块化源码，并新增了**本地简历编辑器**功能。

**项目路径**: `/Users/zane/Downloads/inbox/getOffer`

---

## 2. 目录结构

```
getOffer/
├── original/          # 原始扩展文件（只读参考）
│   ├── manifest.json, background.js, content.js, panel.js, popup.js
│   ├── popup.html, customerService.html, help.html, styles.css
│   ├── images/, _locales/
├── extracted/         # 逆向提取的 webpack 模块（中间产物）
│   ├── background/    # ~15 modules
│   ├── content/       # ~15 modules  
│   └── panel/         # ~140 modules (大部分是 Ant Design)
├── src/               # ★ 重建的源码
│   ├── manifest.json  # 修改后的 manifest（含 editor、tabs 权限）
│   ├── background/    # Service Worker
│   ├── content/       # Content Script（表单填写+面板注入）
│   ├── panel/         # 侧边栏 React UI
│   ├── editor/        # ★ 新增：本地简历编辑器
│   ├── shared/        # 共享模块（config, selectors, resumeModel）
│   └── popup/         # Popup 脚本
├── dist/              # ★ webpack 构建输出（加载到 Chrome 的目录）
├── scripts/           # 逆向辅助脚本
├── webpack.config.js  # 构建配置
├── .babelrc           # Babel 配置（JSX + Chrome 90）
├── package.json       # 依赖配置
└── 简历数据结构.txt      # 用户提供的完整简历 JSON 结构
```

---

## 3. 架构与通信流

```
┌─────────────────────────────────────────────────────────────┐
│ Chrome Extension                                           │
│                                                            │
│  popup.js ──(chrome.tabs.sendMessage)──► content/index.js  │
│                                           │                │
│  background/index.js ◄──(chrome.runtime)──┤                │
│    │                                      │                │
│    ├─ openEditor → chrome.tabs.create     ├─ openPanel()   │
│    ├─ proxyFetch → fetch()                ├─ closePanel()  │
│    ├─ getFieldMappings                    ├─ fillForm()    │
│    └─ getUpdateNotification               │                │
│                                           ▼                │
│                              panel iframe (panel.html)     │
│                              ↕ window.parent.postMessage   │
│                              content/index.js 监听         │
│                                                            │
│  editor.html ──(chrome.storage.local)── 独立页面           │
└─────────────────────────────────────────────────────────────┘
```

**关键通信机制**:
- `popup.js` → content script: `chrome.tabs.sendMessage({action: 'openPanel'})`
- panel iframe ↔ content script: `window.parent.postMessage({type: 'FROM_PANEL', action})` / `window.postMessage({type: 'FROM_CONTENT'})`
- content script → background: `chrome.runtime.sendMessage({action: 'openEditor'})`
- editor: 直接用 `chrome.storage.local` 读写，不经过 message

---

## 4. 构建系统

### 依赖
```json
{
  "dependencies": { "react": "^18", "react-dom": "^18", "antd": "^5", "@ant-design/icons": "^5" },
  "devDependencies": { "webpack": "^5", "webpack-cli": "^6", "babel-loader": "^9", "@babel/core": "^7", "@babel/preset-env": "^7", "@babel/preset-react": "^7", "style-loader": "^4", "css-loader": "^7", "copy-webpack-plugin": "^12" }
}
```

### 命令
```bash
npm install              # 安装（需代理: npm config set proxy http://127.0.0.1:7890）
npm run build            # 生产构建 → dist/
npm run build:dev        # 开发构建
npm run watch            # 监听模式
```

### Webpack 关键配置
- **5 个入口**: background, content, popup, panel, editor
- **splitChunks: false** — 禁用代码拆分，每个入口独立打包（避免 chunk 文件名管理问题）
- **Babel**: `@babel/preset-react` (automatic runtime) + target Chrome 90
- **CopyPlugin**: 从 `src/manifest.json`, `src/panel/panel.html`, `src/editor/editor.html`, `original/` 复制静态文件

---

## 5. 重建的源码模块

### background/ (Service Worker)
| 文件 | 功能 |
|---|---|
| `index.js` | 消息路由、生命周期、定时任务、**openEditor 处理** |
| `resumeService.js` | 简历数据 CRUD（chrome.storage.local） |
| `fieldMappings.js` | 字段映射配置（从 API 获取/缓存） |
| `activityStats.js` | 用户活动统计队列 |
| `updateNotification.js` | 版本更新提示 |

### content/ (Content Script)
| 文件 | 功能 |
|---|---|
| `index.js` | 面板管理、消息监听（chrome.runtime + **window.message 桥**） |
| `formFiller.js` | 核心表单填写引擎（10种输入类型） |
| `utils/domHelper.js` | DOM 操作工具（等待元素、安全点击、可见性判断） |
| `utils/selectHelper.js` | Select/下拉框处理 |
| `utils/dateHelper.js` | 日期选择器处理 |

### panel/ (侧边栏 UI - React + Ant Design)
| 文件 | 功能 |
|---|---|
| `App.jsx` | 主 UI（简历预览、一键填写、FAQ、设置） |
| `messageBridge.js` | **window.parent.postMessage** 通信桥 |
| `hooks.js` | 5 个自定义 Hook（useResumeData, useAutoFill 等） |
| `panel.css` | 面板样式 |
| `panel.html` | iframe 宿主页面 |

### editor/ (★ 新增 - 本地简历编辑器)
| 文件 | 功能 |
|---|---|
| `App.jsx` | 10-tab React 表单编辑器，支持导入/导出 JSON |
| `editor.html` | 编辑器页面 |

### shared/
| 文件 | 功能 |
|---|---|
| `config.js` | 环境检测、API URL 配置 |
| `selectors.js` | CSS 选择器常量（表单、日期、单选等） |
| `selectConfigs.js` | 下拉框配置 |
| `resumeModel.js` | 简历数据模型 |

---

## 6. 踩过的坑

| 问题 | 根因 | 修复 |
|---|---|---|
| 扩展点击后"被 Chrome 屏蔽" | `dist/` 缺少 `panel.html` | 创建 `panel.html` 并加入 `web_accessible_resources` |
| Panel 加载超时 timeout | iframe 内 `window.postMessage` 发给了自己 | 改为 `window.parent.postMessage` |
| Editor 打开空白 | webpack splitChunks 产生的 chunk 未被 HTML 引用 | 禁用 `splitChunks`，每个入口独立打包 |
| npm 安装超慢 | 网络问题 | 通过 Clash 代理: `npm config set proxy http://127.0.0.1:7890` |
| `selectors.js` import 路径错误 | `content/utils/` 需要 `../../shared/` 而非 `../shared/` | 修正相对路径 |
| `npm run build` 删除手动文件 | webpack `clean: true` | 将 manifest, panel.html, editor.html 放入 src/ 通过 CopyPlugin 复制 |

---

## 7. 简历数据结构（完整版）

存储在 `chrome.storage.local` 的 `resume_data` key 下。用户提供的完整结构见 `简历数据结构.txt`。

**核心 sections**: `basicInfo`(30+ fields), `education`, `projectExperience`, `internshipExperience`, `workExperience`, `awards`, `certificates`, `publications`, `patents`, `languageSkills`, `campusActivities`, `campusLeadership`, `familyMembers`, `combinedExperience`, `jobIntention`, `selfIntroduction`, `educationAwards`, `customFields`

---

## 8. 待完成功能

- [ ] **多简历管理** — 支持多份简历切换（用户已提出需求）
  - 存储方案: `resume_list: [{id, name, data}]` + `active_resume_id`
  - 保持 `resume_data` 与活跃简历同步（兼容 content script）
  - UI: 编辑器 Header 添加下拉选择器、新建/复制/删除/重命名
- [ ] 更完善的 formFiller 字段映射（匹配新数据结构的额外字段）
- [ ] 自动化测试验证
