# Offer Knowledge Base

OfferLink Chrome 扩展项目的知识库，包含项目架构、组件实现和开发经验的详细文档。

## 📚 知识条目

| 目录 | 内容 | 说明 |
|------|------|------|
| `offer_field_mapping_shadowing` | 字段映射 Bug 模式 | 服务端缓存的 fieldMappings 遮蔽本地正确映射的 Bug 及修复方案 |
| `offer_generic_interaction` | 通用交互策略 | 框架无关的 UI 交互方案，包括 PopupDetector、GenericInteraction 等 |
| `offer_phoenix_date_picker` | Phoenix 日期选择器 | Phoenix 日历组件的完整实现，包括月选择和日期选择两种模式 |
| `offer_resume_preview` | 简历预览组件 | 简历预览的完整知识，含 iframe 环境问题、项目架构、面板重构等 |
| `offer_select_adapter` | SelectAdapter 架构 | 14 种 UI 框架的下拉选择器适配器工厂模式 |

## 📁 文件结构

每个知识条目包含：
- `metadata.json` — 摘要、来源引用
- `timestamps.json` — 创建/修改时间
- `artifacts/` — 详细文档和实现说明

## 🔗 关联项目

- [getOffer](https://github.com/syntaxsage2/getOffer) — OfferLink Chrome 扩展源码
