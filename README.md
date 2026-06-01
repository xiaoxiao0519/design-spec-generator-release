# 📐 Design Spec Generator — 设计规范文档生成

> AI 驱动的设计规范文档生成工具，从 Figma 文件自动提取设计 Token，输出模块化、AI 可读的 Markdown 规范文档。

设计到开发流程第三步：**检查文件规范 → 评审设计质量 → 生成设计规范 → 验收 UI 实现**。

## ✨ 核心能力

| 能力 | 说明 |
|------|------|
| **13 模块覆盖** | 项目概述、设计原则、色彩、排版、间距、图标、布局、组件、动效、图片、暗黑模式、无障碍、术语表 |
| **模块化输出** | 每个模块独立成章、可增可删、不相互依赖 |
| **AI 可读** | 颜色、组件、布局描述均可作为 AI 生图 prompt 输入 |
| **数据驱动** | 所有规范数据从 Figma API 提取，非凭空编造 |
| **多输入支持** | 项目/页面/画板链接、截图、补充说明 |

## 📁 文件结构

```
design-spec-generator/
├── README.md                              ← 本文件
├── SKILL.md                               ← WorkBuddy 完整版 Skill
├── portable-design-spec-generator.md      ← 跨平台便携版（Cursor / Claude Code / Codex）
├── .gitignore
├── figma-design-spec-generator.md         ← Figma Make 专用版
└── references/
    ├── extraction-guide.md                ← Figma 数据提取指南
    └── modular-template.md                ← 模块化模板结构
```

## 🚀 安装

### WorkBuddy

```bash
git clone https://github.com/YOUR_USERNAME/design-spec-generator.git
cp -r design-spec-generator ~/.workbuddy/skills/design-spec-generator
```

### Cursor

```bash
mkdir -p .cursor/rules
cp portable-design-spec-generator.md .cursor/rules/design-spec-generator.mdc
```

### Claude Code

```bash
cat portable-design-spec-generator.md >> CLAUDE.md
```

### Figma Make

将 `figma-community-skill/SKILL.md` 上传到 Figma Make skill。

## 📖 使用方式

**触发词**（任一即可）：
- "帮我生成设计规范" / "输出设计文档"
- "Figma转规范" / "写个设计规范"
- "生成组件文档" / "设计交付"
- "设计系统文档"

**输入**：Figma 项目/页面/画板链接（推荐） / 设计稿截图

## 🔗 配套 Skill

| Skill | 说明 |
|-------|------|
| [design-file-audit](https://github.com/YOUR_USERNAME/design-file-audit) | 设计文件规范检查 |
| [design-review](https://github.com/YOUR_USERNAME/design-review) | 设计评审（交互/视觉质量） |
| [ui-acceptance-checker](https://github.com/YOUR_USERNAME/ui-acceptance-checker) | UI 验收（Figma vs 实现对比） |

## 📄 License

MIT
