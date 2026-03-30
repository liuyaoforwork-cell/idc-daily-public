# IDC 行业情报归档站 — 开发实践记录

> 项目名称：idc-daily-public  
> 开发日期：2026-03-30  
> 部署地址：https://liuyaoforwork-cell.github.io/idc-daily-public/  
> 仓库地址：https://github.com/liuyaoforwork-cell/idc-daily-public

---

## 目录

1. [项目概述](#1-项目概述)
2. [架构设计](#2-架构设计)
3. [开发历程（8 次迭代）](#3-开发历程)
4. [技术方案详解](#4-技术方案详解)
5. [踩坑与教训](#5-踩坑与教训)
6. [文件清单与说明](#6-文件清单与说明)
7. [运维指南](#7-运维指南)
8. [未来扩展方向](#8-未来扩展方向)

---

## 1. 项目概述

### 1.1 背景

IDC（数据中心）行业情报日报是内部自动化流程的产出物。为了让研究成果能以更友好的方式对外分享，需要一个**公开版归档站**——去除内部敏感信息，保留核心情报价值，并以优质的阅读体验呈现。

### 1.2 目标

- ✅ 永久可访问的公开 URL（无密码、HTTPS）
- ✅ 优秀的阅读体验（深浅主题、搜索、板块导航）
- ✅ 桌面端 + 手机端双版本
- ✅ 零后端依赖（纯静态、GitHub Pages 托管）
- ✅ 新期刊发布流程极简（新建 HTML → 更新数组 → git push）

### 1.3 最终成果

| 页面 | URL | 说明 |
|------|-----|------|
| 📰 归档站首页 | `/index.html` | 期刊列表 + 情报搜索 + 最新期刊全文 + 统计数据 |
| 📖 期刊详情页 | `/issues/2026-03-30.html` | 完整情报内容 + 阅读进度条 + 板块导航 + 图表 |
| 📱 手机版 | `/mobile.html` | 专为小屏优化的 Tab 导航界面 |

---

## 2. 架构设计

### 2.1 技术栈

```
前端: 原生 HTML + CSS + JavaScript（零框架、零构建工具）
部署: GitHub Pages（legacy 模式，main 分支根目录）
版本控制: Git
数据: 内联在 HTML 的 JS 变量中（ISSUES / INTEL_ITEMS 数组）
```

### 2.2 选择纯静态方案的原因

| 考虑因素 | 决策 |
|---------|------|
| 受众规模 | 小范围分享，不需要 CMS |
| 更新频率 | 每日/每周一期，手动更新可接受 |
| 维护成本 | 零服务器、零数据库、零 CI/CD |
| 持久性 | GitHub Pages 永久免费 |
| 性能 | 单文件加载，无网络请求（数据内联） |

### 2.3 文件结构

```
idc-daily-public/
├── index.html          # 归档站首页（776 行, 40KB）
├── mobile.html         # 手机端专属页面（747 行, 40KB）
├── site.json           # 站点配置元数据
├── .nojekyll           # 跳过 GitHub Pages 的 Jekyll 处理
├── DEV-PRACTICE.md     # 本文档
├── issues/
│   └── 2026-03-30.html # 期刊详情页（676 行, 36KB）
└── .git/
```

### 2.4 数据流

```
内部自动化情报 → 人工审核/脱敏 → 编写公开版 Markdown
    → 制作 HTML 详情页（issues/YYYY-MM-DD.html）
    → 更新首页 ISSUES + INTEL_ITEMS 数组
    → git push → GitHub Pages 自动部署（~60s）
```

---

## 3. 开发历程

整个项目在 **2026-03-30 一天内** 经历了 **8 次迭代提交**，从单页 MVP 演进为功能完整的归档站。

### 迭代时间线

| # | 提交 | 里程碑 | 关键变化 |
|---|------|--------|----------|
| 1 | `1fb65ec` | 🚀 MVP 上线 | 单页 HTML，深色主题，展示当日情报 |
| 2 | `5b5d664` | 📚 归档站架构 | 拆分为首页 + 详情页，支持多期刊 |
| 3 | `93fb551` | 🔧 修复部署 | 添加 `.nojekyll` 解决 GitHub Pages 路径问题 |
| 4 | `54f7849` | 🔍 搜索升级 | 从期刊级搜索升级为情报条目级搜索 |
| 5 | `dde77cf` | ☀️ 浅色主题 | 默认改为浅色 + 首页要闻区 |
| 6 | `0b08930` | 🎨 主题彻底修复 | CSS 变量反转架构 + 首页全文展示 |
| 7 | `7c62138` | 🐛 紧急 Bug 修复 | 中文引号导致 JS 语法错误 |
| 8 | `ab57e4f` | 📱 手机版 | 新增专属手机端页面 |

### 3.1 迭代 1：MVP 单页

**目标**：最快速度上线一个可分享的公开版链接。

- 深色风格单页 HTML
- 所有内容内联
- 尝试 Netlify 匿名部署 → 需要密码，弃用
- 最终选定 GitHub Pages：免费、永久、HTTPS、无密码

### 3.2 迭代 2：归档站架构

**目标**：支持多期刊归档，而非一次性单页。

- 首页 `index.html`：期刊列表 + 搜索 + 暗亮切换 + 统计
- 详情页 `issues/YYYY-MM-DD.html`：完整情报 + 阅读进度条 + 板块导航 pill + 招投标柱状图 + 分享/打印 + print CSS
- 数据结构：`ISSUES` 数组管理期刊元数据

### 3.3 迭代 3：部署修复

**问题**：GitHub Pages 默认启用 Jekyll，会忽略下划线开头的文件/目录。  
**修复**：根目录添加 `.nojekyll` 空文件。

### 3.4 迭代 4：搜索升级

**用户反馈**：搜索只能到期刊级别，无法直接找到某条具体情报。

- 新增 `INTEL_ITEMS` 数组：内联所有情报条目（标题、正文、标签、来源、板块锚点）
- 搜索支持匹配情报标题、正文、标签、来源
- 双模式搜索：「📰 情报条目」+「📚 按期刊」可切换
- 搜索结果：高亮关键词、显示摘要、跳转详情页

### 3.5 迭代 5：默认浅色 + 要闻区

**用户偏好**：浅色为默认模式更舒适。

- 默认主题从深色改为浅色
- 首页新增「📢 本期要闻」摘要卡片区

### 3.6 迭代 6：主题系统彻底重构

**根本问题**：之前 CSS `:root` 定义深色变量，`[data-theme="light"]` 覆盖浅色——但 `localStorage` 缓存导致主题切换不可靠。

**解决方案（CSS 变量反转）**：
```
:root           → 浅色默认值
[data-theme="dark"] → 深色覆盖值
```
- 浅色模式：不设 `data-theme` 属性（使用 `:root` 默认）
- 深色模式：设置 `data-theme="dark"`
- 防 FOUC：`<head>` 中内联 `<script>` 在 body 渲染前读取 `localStorage`
- 首页改版：从摘要卡片改为展示最新一期完整情报（按板块分组）

### 3.7 迭代 7：紧急 Bug 修复 🔥

**现象**：页面上线后完全空白，控制台报 `SyntaxError: Unexpected identifier`。

**根因**：`INTEL_ITEMS` 数据中包含中文双引号 `""` (U+201C/U+201D)，在 JS 双引号字符串 `"..."` 中被解析为字符串终止符 → 整个 `<script>` 块无法执行。

**修复**：将所有中文引号替换为角引号 `「」`（共 8 处）。

**教训**：
> ⚠️ 往 JS 字符串字面量中写入中文内容时，必须确保内部引号不与外部定界符冲突。最佳实践：用 `JSON.stringify()` 处理或统一用角引号。

### 3.8 迭代 8：手机端专属页面

**目标**：为手机用户提供原生级别的阅读体验。

- 底部 Tab 导航：📰 最新、📚 期刊、📊 统计
- 最新视图：hero 摘要 → 横向滑动板块筛选 pill → 情报卡片列表（展开/收起）
- 期刊视图：紧凑列表卡片
- 统计视图：4 宫格 + 板块分布进度条 + 热门标签云
- 顶部搜索 + 暗色模式（与桌面版共享 `localStorage`）
- 安全区适配（刘海屏/底部手势条）
- 双向互联：手机版 ↔ 桌面版互链

---

## 4. 技术方案详解

### 4.1 主题切换系统

```css
/* 浅色默认 */
:root {
  --bg: #f5f5f7;
  --card: #ffffff;
  --text: #1d1d1f;
  /* ... */
}

/* 深色覆盖 */
[data-theme="dark"] {
  --bg: #0a0a0c;
  --card: #1c1c1e;
  --text: #f5f5f7;
  /* ... */
}
```

**防 FOUC 脚本**（放在 `<head>` 中）：
```html
<script>
  var t = localStorage.getItem('theme');
  if (t === 'dark') document.documentElement.setAttribute('data-theme', 'dark');
</script>
```

### 4.2 数据内联策略

所有数据以 JS 变量形式内联在 HTML 中：

```javascript
// 期刊列表
const ISSUES = [
  { date: "2026-03-30", title: "...", summary: "...", count: 12, url: "issues/2026-03-30.html" }
];

// 情报条目（用于搜索和首页展示）
const INTEL_ITEMS = [
  { section: "重大信号", title: "...", body: "...", tags: ["算力","投资"], source: "...", sourceUrl: "...", insight: "..." }
];
```

**优点**：零 API 请求、无 CORS 问题、离线可用  
**缺点**：每次更新需手动编辑 HTML

### 4.3 搜索实现

纯前端内存搜索，对 `INTEL_ITEMS` / `ISSUES` 数组做字符串匹配：

```javascript
function searchIntelItems(query) {
  const q = query.toLowerCase();
  return INTEL_ITEMS.filter(item =>
    item.title.toLowerCase().includes(q) ||
    item.body.toLowerCase().includes(q) ||
    item.tags.some(t => t.toLowerCase().includes(q)) ||
    item.source.toLowerCase().includes(q)
  );
}
```

搜索结果高亮使用正则替换：
```javascript
function highlight(text, query) {
  return text.replace(new RegExp(query, 'gi'), '<mark>$&</mark>');
}
```

### 4.4 手机端 Tab 导航

采用底部固定 Tab 栏 + 视图切换模式：

```javascript
function switchTab(tabName) {
  // 隐藏所有视图
  ['latest', 'issues', 'stats'].forEach(v => {
    document.getElementById('view-' + v).style.display = 'none';
  });
  // 显示目标视图
  document.getElementById('view-' + tabName).style.display = '';
  // 更新 Tab 按钮高亮
}
```

底部安全区适配：
```css
.tab-bar {
  padding-bottom: max(8px, env(safe-area-inset-bottom));
}
```

### 4.5 阅读进度条（详情页）

```javascript
window.addEventListener('scroll', () => {
  const scrollTop = document.documentElement.scrollTop;
  const scrollHeight = document.documentElement.scrollHeight - window.innerHeight;
  const progress = (scrollTop / scrollHeight) * 100;
  document.getElementById('progress-bar').style.width = progress + '%';
});
```

### 4.6 打印样式

详情页支持浏览器打印 / 导出 PDF：

```css
@media print {
  .topbar, .section-nav, .back-to-top, .progress-wrap { display: none; }
  body { background: white; color: black; }
  .card { break-inside: avoid; border: 1px solid #ddd; }
}
```

---

## 5. 踩坑与教训

### 5.1 🔥 中文引号导致 JS 崩溃（P0 Bug）

**严重程度**：页面完全空白  
**原因**：中文内容中的 `""` 在 JS 双引号字符串中被误解析为定界符  
**修复**：统一使用角引号 `「」` 替代  
**预防**：
- 对动态内容使用 `JSON.stringify()` 
- 制定编码规范：JS 字符串中不使用中文引号

### 5.2 GitHub Pages Jekyll 处理

**现象**：部分路径 404  
**原因**：GitHub Pages 默认启用 Jekyll  
**修复**：根目录添加 `.nojekyll`

### 5.3 主题切换闪烁（FOUC）

**现象**：页面加载时先显示默认主题，再闪到用户偏好主题  
**原因**：JS 在 DOM 加载后才执行  
**修复**：在 `<head>` 中添加内联阻塞脚本，渲染前设置主题

### 5.4 CSS 变量覆盖方向

**教训**：当默认主题是浅色时，`:root` 应该定义浅色值，`[data-theme="dark"]` 覆盖深色值。反过来会导致 `localStorage` 缓存与 CSS 默认值冲突。

### 5.5 Netlify 匿名部署的局限

**尝试**：Netlify Drop（匿名部署）  
**问题**：匿名站点有密码保护提示  
**弃用原因**：目标是无障碍公开访问  
**替代**：GitHub Pages（完全免费、无密码、永久有效）

---

## 6. 文件清单与说明

| 文件 | 行数 | 大小 | 说明 |
|------|------|------|------|
| `index.html` | 776 | 40KB | 归档站首页：期刊列表 + 条目级搜索 + 最新期刊全文 + 统计 |
| `mobile.html` | 747 | 40KB | 手机端页面：底部 Tab 导航 + 板块筛选 + 标签云 |
| `issues/2026-03-30.html` | 676 | 36KB | 首期详情页：完整情报 + 进度条 + 导航 + 图表 |
| `site.json` | 16 | <1KB | 站点配置元数据 |
| `.nojekyll` | 0 | 0 | 跳过 Jekyll 处理 |
| `DEV-PRACTICE.md` | - | - | 本开发实践文档 |

**总计**：~2200 行代码，~120KB（不含 `.git`）

---

## 7. 运维指南

### 7.1 发布新期刊

```bash
# 1. 在 issues/ 创建新的 HTML（复制 2026-03-30.html 为模板）
cp issues/2026-03-30.html issues/2026-03-31.html

# 2. 编辑新文件，替换情报内容

# 3. 更新 index.html 的 ISSUES 和 INTEL_ITEMS 数组

# 4. 更新 mobile.html 的 ISSUES 和 INTEL_ITEMS 数组

# 5. 提交并推送
git add -A
git commit -m "发布 2026-03-31 期刊"
git push origin main

# GitHub Pages 约 60 秒后自动部署
```

### 7.2 修改主题颜色

编辑 CSS 变量即可全局生效：
- `:root { ... }` — 浅色主题色值
- `[data-theme="dark"] { ... }` — 深色主题色值

### 7.3 部署配置

- **分支**：`main`
- **部署目录**：根目录 `/`
- **模式**：GitHub Pages legacy（Settings → Pages → Source: main branch）
- **必须文件**：`.nojekyll`

---

## 8. 未来扩展方向

| 方向 | 描述 | 优先级 |
|------|------|--------|
| 自动化发布 | CI/CD 流水线自动生成新期刊 HTML | ⭐⭐⭐ |
| 数据外置 | 将 INTEL_ITEMS 抽取为 JSON 文件，减少重复 | ⭐⭐⭐ |
| RSS 订阅 | 生成 RSS feed 供阅读器订阅 | ⭐⭐ |
| PWA 支持 | Service Worker + manifest，支持"添加到主屏幕" | ⭐⭐ |
| 评论互动 | 集成 Giscus / Utterances 评论系统 | ⭐ |
| 全文索引 | 使用 Lunr.js / FlexSearch 替代简单字符串匹配 | ⭐ |

---

## 附录：技术决策记录

| 决策 | 选项 | 最终选择 | 理由 |
|------|------|---------|------|
| 前端框架 | React / Vue / 原生 | 原生 HTML+CSS+JS | 零构建、零依赖、单文件即可部署 |
| 部署平台 | Netlify / Vercel / GitHub Pages | GitHub Pages | 永久免费、无密码限制、与代码仓库集成 |
| 数据管理 | API / JSON 文件 / 内联 | JS 变量内联 | 零网络请求、离线可用、KISS 原则 |
| 搜索方案 | Algolia / Lunr.js / 原生 | 原生字符串匹配 | 数据量小（<100 条），无需全文索引引擎 |
| 主题方案 | CSS 媒体查询 / JS 切换 | CSS 变量 + JS + localStorage | 用户可手动切换，偏好持久化 |

---

*文档编写于 2026-03-30，记录 IDC 行业情报归档站从零到完成的完整开发实践。*
