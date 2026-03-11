# OfferLink 通用交互策略

## 问题背景

OfferLink 原项目为 9 个 UI 框架（Phoenix、Ant Design、Element UI、Layui、Kuma 等）维护了 **335 个 CSS 选择器 / ~4300 行适配代码**。每新增一个框架需要完整的 Adapter 和选择器集。

项目有两条独立的痛点线：
1. **字段映射**：不同网站 label 写法不同（"入学时间" vs "开始时间" vs "就读起始时间"）
2. **值交互**：不同框架的日期选择器/下拉选择器内部 DOM 结构不同

## 解决方案

### 值交互：混合通用策略（已实现）

**分支**: `feature/generic-interaction`

已知框架 → CSS 选择器快速通道（保留现有代码）
未知框架 → 文本语义通用流程

#### 核心工具

| 文件 | 核心能力 |
|------|---------|
| `popupDetector.js` | `waitForPopup(triggerFn)` — MutationObserver 检测弹窗（position/z-index/面积判定） |
| `genericInteraction.js` | `findByText(container, pattern)` — 按文本/正则找元素 |
| | `findNavButtons(container)` — 识别 ‹›«» 箭头导航按钮 |
| | `navigateCalendar(popup, {year, month, day})` — 通用日历导航 |
| | `selectOption(popup, targetText)` — 通用下拉选项匹配 |

#### 日期选择器 3 步流程

```
Step 1: tryKeyboardDate()       — 模拟键盘输入（非 readonly 时）
Step 2: tryPopupCalendarDate()  — PopupDetector + navigateCalendar（按文本找年/月/日）
Step 3: _forceSetInputValue()   — 强制设值（兜底）
```

### 字段映射：LLM 批量映射（方案待实施）

扫描 DOM 提取所有 `{section, label, type}` + 简历 schema → 一次 LLM 调用 → 返回完整映射表 → 缓存到 `chrome.storage.local`。

无 API key 时 fallback 到现有硬编码同义词表。

## agent-browser 参考

Vercel 的 [agent-browser](https://github.com/vercel-labs/agent-browser) 用 Playwright `ariaSnapshot()` 获取无障碍树 + ref 系统，完全跨框架。核心启示：

- 按 **ARIA role + text** 而非 CSS class 定位元素
- `findCursorInteractiveElements()` 检测 `cursor:pointer` / `onclick` 非标准交互元素
- LLM 消费 snapshot → 决策操作 → ref 执行，工具层不做决策

Chrome 扩展中无法使用 Playwright，但其"按语义而非 class 操作"的思路直接启发了 `genericInteraction.js` 的设计。
