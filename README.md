# design-skill

对话式 UI 原型生成 skill —— 通过自然语言描述即可生成完整 HTML/CSS 界面，与 Claude Code 配合使用。

## 安装

将 `skills/design/` 目录复制到你的 Claude Code 项目：

```bash
# 方式一：直接使用 git clone
git clone https://github.com/owlteam990/design-skill.git .claude

# 方式二：只复制 skill 文件
mkdir -p .claude/skills/design
cp skills/design/SKILL.md .claude/skills/design/
```

Claude Code 会自动发现 `.claude/skills/` 下的 skill。

## 使用

在 Claude Code 对话中直接输入：

```
/design
```

然后描述你想要的界面：

```
设计一个 SaaS 登录页面，白色背景，蓝色主色，左侧品牌标语右侧表单
```

Claude 会生成完整自包含的 HTML/CSS 设计稿，你可以：
- 复制到浏览器预览
- 在 iframe 中内嵌渲染
- 通过 CSS 变量调整主色、字体、间距
- 导出为 HTML 或 PDF

## 功能

| 功能 | 说明 |
|------|------|
| 对话式生成 | 自然语言描述 → 完整 HTML+CSS |
| 品牌提取 | 从项目中提取颜色/字体 token |
| 内联编辑 | 通过 CSS 变量实时调整设计参数 |
| 导出 | HTML 文件下载 / 浏览器打印 PDF |

## 设计规范

Skill 内置了 6 条设计规范，确保生成质量：

1. CSS Custom Properties — 必须定义 `--primary-color`、`--font-family`、`--spacing-unit`
2. 自包含 — 所有样式内嵌，不依赖外部资源
3. 完整替换 — 每次输出完整 HTML，不是 diff
4. 响应式 — 适配 375px / 768px / 1280px
5. 现代美学 — 干净、极简、专业
6. 无网络请求 — 安全沙箱兼容

## 目录结构

```
.claude/
├── README.md
└── skills/
    └── design/
        └── SKILL.md   # skill 定义文件
```

## License

MIT
