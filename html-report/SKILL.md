# HTML Report Skill

Generate standalone HTML report files with strict visual and structural constraints.

## Triggers

- 用户要求生成 HTML 报告、项目总结、进度汇报、技术文档
- 用户说"总结成 HTML"、"做个报告"、"生成报告"

## Constraints (必须遵守)

### 1. 颜色管理
- **最多三种颜色**：主色（accent）、背景色（bg/surface）、文字色（text）
- **禁止使用渐变色**（linear-gradient、radial-gradient 等一律禁止）
- 通过 opacity / lighten / darken 在三种颜色内派生层次感

### 2. ICON 使用
- **尽可能不使用 ICON**
- 如必须使用，只用**单色 ICON**（lucide、heroicons outline 等）
- 禁止彩色 ICON、多色 ICON、SVG 渐变 ICON

### 3. 侧边栏
- **必须有侧边栏**，用于报告内容定位导航
- 侧边栏固定定位（position: fixed），滚动时高亮当前章节
- 点击侧边栏链接需**立即切换高亮**（不依赖 IntersectionObserver 延迟）
- 侧边栏底部放置主题切换按钮

### 4. 双色主题
- **必须支持暗色和亮色两种主题**
- 默认跟随系统 `prefers-color-scheme`
- 提供手动切换按钮
- 使用 CSS 变量 + `[data-theme="dark"]` 切换

### 5. 语言
- **全部使用中文输出**
- 专有名词和技术术语保留英文（如 MCU、HID、LVGL、TypeScript 等）
- 代码块、命令、文件路径保留英文

### 6. 字体
- **主字体：Maple Mono NF CN**
- 字体文件位于 `tools/fonts/MapleMono-NF-CN-Regular.ttf`，通过 `@font-face` 本地引入
- 回退字体：`-apple-system, "Segoe UI", sans-serif`
- 代码字体：`'Maple Mono NF CN', "SF Mono", Consolas, monospace`

## Color Palette Template

```css
/* 亮色主题 */
:root {
  --bg: #f5f2ed;        /* 暖灰白背景 */
  --surface: #ebe7e0;   /* 卡片/侧边栏背景 */
  --text: #1c1917;      /* 主文字 */
  --text-muted: #78716c; /* 次要文字 */
  --accent: #92400e;    /* 强调色（琥珀棕） */
  --border: #d6d3d1;    /* 边框 */
}

/* 暗色主题 */
[data-theme="dark"] {
  --bg: #1a1714;
  --surface: #242019;
  --text: #e7e5e4;
  --text-muted: #a8a29e;
  --accent: #d97706;
  --border: #3a352e;
}
```

## HTML Structure Template

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{报告标题}</title>
  <style>
    /* 字体引入 */
    @font-face {
      font-family: 'Maple Mono NF CN';
      src: url('../tools/fonts/MapleMono-NF-CN-Regular.ttf') format('truetype');
      font-weight: 400;
      font-style: normal;
      font-display: swap;
    }
    /* CSS 变量（亮色 + 暗色） */
    /* 侧边栏样式 */
    /* 主内容样式 */
    /* 表格、卡片、标签、代码块样式 */
    /* 响应式：移动端隐藏侧边栏 */
  </style>
</head>
<body>
  <!-- 侧边栏 -->
  <aside class="sidebar">
    <div class="sidebar-title">{项目名}</div>
    <nav>
      <a href="#section-id">章节名称</a>
      ...
    </nav>
    <div class="sidebar-footer">
      <button class="theme-toggle" onclick="toggleTheme()">暗色 / 亮色</button>
    </div>
  </aside>

  <!-- 主内容 -->
  <div class="main">
    <h1>{报告标题}</h1>
    <p class="subtitle">{副标题}</p>

    <h2 id="...">章节标题</h2>
    <p>正文内容</p>
    <ul><li>列表项</li></ul>
    <table>...</table>
    <pre><code>代码块</code></pre>
    <div class="card">卡片内容</div>
  </div>

  <script>
    // 主题切换
    // 侧边栏点击立即高亮
    // 滚动同步高亮（IntersectionObserver）
    // 系统深色模式跟随
  </script>
</body>
</html>
```

## Key Implementation Notes

### 侧边栏高亮（必须同时实现两种）
```javascript
// 1. 点击立即高亮
navLinks.forEach(link => {
  link.addEventListener('click', () => {
    navLinks.forEach(l => l.classList.remove('active'));
    link.classList.add('active');
  });
});

// 2. 滚动同步高亮
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      navLinks.forEach(l => l.classList.remove('active'));
      document.querySelector(`.sidebar nav a[href="#${entry.target.id}"]`)
        ?.classList.add('active');
    }
  });
}, { rootMargin: '-20% 0px -70% 0px' });
```

### 响应式处理
```css
@media (max-width: 768px) {
  .sidebar { display: none; }
  .main { margin-left: 0; padding: 20px; }
}
```

### 进度条（不依赖任何库）
```html
<div class="progress-bar">
  <div class="progress-fill" style="width: 75%"></div>
</div>
<p class="progress-label">75% 完成</p>
```

## Checklist

生成报告前逐项检查：

- [ ] 颜色是否不超过三种？
- [ ] 是否有渐变色？（必须移除）
- [ ] 是否使用了 ICON？（尽可能移除，必须用时确认是单色）
- [ ] 是否有侧边栏？
- [ ] 侧边栏点击是否立即高亮？
- [ ] 是否支持暗色/亮色切换？
- [ ] 是否跟随系统主题偏好？
- [ ] 内容是否全部中文？（专有名词除外）
- [ ] 字体是否为 Maple Mono NF CN？
- [ ] 是否为单个独立 HTML 文件？（无外部依赖，字体 CDN 除外）
