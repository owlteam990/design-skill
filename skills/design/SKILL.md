---
name: claude-design
description: 对话式 UI 原型生成。通过自然语言描述即可生成完整 HTML/CSS 界面，实时预览、内联编辑设计 token、提取品牌色彩系统、导出 HTML/PDF。
when_to_use: 用户需要生成 UI 原型、设计界面、创建 landing page、绘制组件、提取品牌色、调整设计 token 时使用。触发词："design"、"设计"、"原型"、"prototype"、"UI"、"landing page"、"dashboard"、"界面"、"品牌色"、"create a design"。
user-invocable: true
disable-model-invocation: false
---

# Claude Design — 对话式 UI 原型生成器

通过自然语言对话生成可预览、可编辑、可导出的 HTML/CSS 界面原型。

## 使用方式

1. 用户描述想要的界面（中英文均可）
2. Claude 生成完整 HTML/CSS 设计稿
3. 用浏览器打开预览，或用 iframe srcdoc 内嵌渲染
4. 通过 CSS 变量实时调整主色、字体、间距
5. 导出为 HTML 文件或 PDF

## 生成 UI 必须遵循的规范

每个包含设计的回复，必须输出一个完整、自包含的 HTML 文档，放在 `html` 代码块中：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Design</title>
  <style>
    :root {
      --primary-color: #8F482F;
      --font-family: 'Inter', 'Segoe UI', sans-serif;
      --spacing-unit: 16px;
    }
    /* 所有样式使用这些变量 */
  </style>
</head>
<body>
  <!-- 完整设计 -->
</body>
</html>
```

### 规则

1. **CSS Custom Properties** — 必须在 `:root` 中定义 `--primary-color`、`--font-family`、`--spacing-unit`，并在 CSS 中使用它们。这些变量会被控制面板实时覆盖。
2. **自包含** — 所有 CSS 嵌入 `<style>` 标签，不引用外部样式表或 CDN。
3. **完整替换** — 每次设计输出必须是完整 HTML 文档，不是片段或 diff。
4. **响应式** — 至少适配 375px（手机）、768px（平板）、1280px（桌面）三个断点。使用 CSS Grid 或 Flexbox。
5. **现代美学** — 干净、极简、专业。良好排版、充裕留白、微妙阴影。
6. **无网络请求** — 可包含装饰性 JS 但不能发起网络调用。

### 对话模式

- 用户描述需求后立即生成设计，不要问"要我生成吗"
- 修改请求时，输出完整更新后的 HTML，不是补丁
- 如果需求不明确，最多问一个问题然后生成
- 首次使用时简短问候并询问想设计什么

---

## 品牌色彩提取

可以读取项目代码库自动提取设计 token。

### 执行步骤

1. 读取项目中的 CSS / SCSS / Tailwind 配置 / design system 文件 / package.json
2. 列出找到的十六进制颜色码和字体名称
3. 用提取的品牌色生成一张 demo 卡片作为验证

### 提取提示词模板

```
请阅读项目代码库，提取设计 token。查找 CSS、SCSS、Tailwind 配置、
design system 文件、package.json 中的 UI 库线索。

列出：
1. 十六进制颜色码（每行一个，如 #1A2B3C）
2. 字体名称（每行一个，如 Inter）

然后用这些品牌色和字体生成一张 demo 卡片设计。
```

---

## 内联编辑系统

生成的 HTML 中的 CSS 变量可通过外部注入实时覆盖，无需重新生成：

```javascript
// 向 iframe 注入覆盖样式
const style = document.createElement('style')
style.textContent = `:root {
  --primary-color: ${newColor} !important;
  --font-family: ${newFont} !important;
  --spacing-unit: ${newSpacing}px !important;
}`
iframe.contentDocument.head.appendChild(style)
```

控制面板提供三个控件：
- **Primary Color** — 颜色选择器（`<input type="color">`）
- **Font** — 字体下拉（Inter / Manrope / Georgia / JetBrains Mono / System UI）
- **Spacing** — 间距滑块（8px–32px，步长 2px）

---

## HTML 提取

从消息流中提取 HTML 设计稿的正则：

```javascript
// 从流式文本中提取 html 代码块
const match = content.match(/```html\s*([\s\S]*?)```/)
const html = match?.[1]?.trim() ?? null

// 从历史消息中提取最新 html 块（倒序查找）
function extractLatestHtml(messages) {
  for (let i = messages.length - 1; i >= 0; i--) {
    if (messages[i].type !== 'assistant') continue
    const match = messages[i].content.match(/```html\s*([\s\S]*?)```/)
    if (match) return match[1].trim()
  }
  return ''
}
```

---

## 导出

**HTML 导出：** Blob + URL.createObjectURL + 触发 `<a>` 下载

```javascript
const blob = new Blob([htmlContent], { type: 'text/html' })
const url = URL.createObjectURL(blob)
const a = document.createElement('a')
a.href = url
a.download = `design-${Date.now()}.html`
a.click()
URL.revokeObjectURL(url)
```

**PDF 导出：** 调用 iframe 的 `contentWindow.print()` 利用浏览器打印到 PDF。

---

## 完整工作流示例

```
用户: "设计一个赛博朋克风格的贪吃蛇游戏封面"
  → Claude 输出完整 HTML（暗色背景、霓虹边框、故障效果标题）
  → iframe srcdoc 实时渲染
  → 用户可在控制面板切换主色为 #00FF41（霓虹绿）

用户: "把标题字体改大一点，背景加一些网格线"
  → Claude 输出更新后的完整 HTML
  → 预览即时更新
  → 满意后点击导出 HTML 或打印 PDF
```

## 实施建议

- **预览层** — iframe `srcdoc` 属性直接渲染，`sandbox="allow-scripts allow-same-origin"` 保证安全
- **流式更新** — 边接收边提取 html 代码块，提取到就立即更新预览
- **状态管理** — 保存 htmlContent、cssOverrides、brandTokens 三个核心字段
- **品牌提取** — 发送提取提示词后，用正则 `/#[0-9A-Fa-f]{6}\b/g` 提取颜色，用引号匹配提取字体名
