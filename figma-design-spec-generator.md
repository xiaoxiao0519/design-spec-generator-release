---
name: design-spec-generator
description: >
  Generate structured design specification documents from Figma files.
  Covers 13 modular chapters: project overview, design principles, color
  system, typography, spacing and corner radius, icon system, layout and
  grid, component specifications, animation, images and illustrations,
  dark mode, accessibility, and glossary. Extracts design tokens, style
  data, and component definitions from Figma files and outputs a
  comprehensive, AI-readable Markdown specification document ready for
  handoff and AI image generation.
---

# 设计规范文档生成 — Figma Make 专用版

> **两种使用方式**：
> - **上传**：将 `figma-community-skill` 文件夹上传到 Figma Community Skills 平台
> - **粘贴**：打开本文件 → 全选复制 → 粘贴到 Figma Make 对话框即可
>
> Figma Make 拥有当前 Figma 文件的直接访问能力，无需任何 API 配置。

从 Figma 设计文件提取数据，生成结构化、模块化、AI 可读的设计规范 Markdown 文档。

---

## 文档生成原则

- **模块化**：13 个模块独立成章，可增可删，不相互依赖
- **可填充**：模板预留结构化位置，团队成员可直接补充内容
- **AI 可读**：颜色、组件、布局描述均以 AI 可理解方式呈现
- **数据驱动**：所有规范数据从 Figma 文件中提取，非凭空编造
- **可追溯**：关键数据标注来源（Figma Style 名称、组件路径）

---

## Figma 文件结构导航

按 Figma 原生层级遍历提取数据：

```
Document
 └─ Page[] 页面列表
     ├─ 组件库页（📦 Components）→ 提取主组件、Variants
     ├─ 规范页（📐 Style Guide）→ 提取设计 Token 定义
     ├─ 迭代页（🎨 YYYY.MM Vx.x 名称）→ 提取实际应用数据
     └─ 废弃页（🗂 Archive）→ 跳过
         └─ Frame[] / Component[] / Group[]
             └─ 子节点嵌套树
```

**节点关键属性速查**：

| 属性 | 用途 | 取值示例 |
|------|------|----------|
| `name` | 图层名称 | "主按钮"、"Frame 1" |
| `type` | 节点类型 | FRAME, TEXT, COMPONENT, INSTANCE |
| `fills` | 填充色数组 | `[{type:"SOLID", color:{r,g,b}}]` |
| `fillStyleId` | Color Style ID | `"S:xxx"` 或空 |
| `fontName` | 字体 | `{family:"PingFang SC", style:"Medium"}` |
| `fontSize` | 字号 px | 12, 14, 24 |
| `lineHeight` | 行高 | `{value:20, unit:"PIXELS"}` |
| `textStyleId` | Text Style ID | `"S:xxx"` 或空 |
| `cornerRadius` | 圆角 px | 8, 12 |
| `layoutMode` | Auto Layout 模式 | "NONE"/"HORIZONTAL"/"VERTICAL" |
| `paddingLeft/Right/Top/Bottom` | 内边距 | 16, 24 |
| `itemSpacing` | AL 间距 | 8, 12 |
| `layoutSizingHorizontal` | 水平尺寸 | "FIXED"/"HUG"/"FILL" |
| `layoutGrids` | 栅格 | `[{pattern:"COLUMNS", count:12, gutterSize:16}]` |
| `componentId` | 实例组件 ID | — |
| `children` | 子节点数组 | — |

---

## 链接类型识别

| 链接类型 | URL 特征 | 提取范围 |
|----------|----------|----------|
| 项目链接 | 无 `node-id` | 全部页面 |
| 页面链接 | `node-id` 存在，`t` 末尾 `-1` | 指定页面 + Style 面板 |
| 画板链接 | `node-id` 存在，`t` 末尾 `-4` | 指定画板 + Style 面板 |

> 层级优先级：用户语言指令 > Figma URL > 默认项目级

---

## 工作流程

### 步骤 0：获取 Figma Variables（优先）

通过 Figma Variables API 获取已定义的变量。若色值匹配到 Variable，Token 名称使用 Variable 名称（如 `brand/primary`），来源标注"Figma Variable"。未匹配的高频色值（≥10 次出现）记录到附录"待补充为 Variable"。

### 步骤 1：提取设计数据

1. **页面扫描**：识别各页面类型（组件库/规范页/迭代页/废弃页）
2. **Style 面板**：提取 Color/Text/Effect Styles（文件级全量）
3. **组件数据**：提取 Components、Variants、实例数量
4. **布局信息**：提取 Auto Layout 参数、Grid、间距模式
5. **设计 Token**：提取 Spacing、Radius、Opacity 等

### 步骤 2：分析与归类

| Figma 数据 | 归入模块 | 处理方式 |
|------------|----------|----------|
| Color Styles + Variables | M03 色彩系统 | Variable 优先，按命名前缀归类品牌色/语义色/中性色 |
| Text Styles | M04 排版系统 | 按字号排序建立梯度 |
| Auto Layout gap/padding | M05 间距与圆角 | 统计间距值分布，识别 4px 梯度 |
| 组件 + Variants | M08 组件规范 | 提取变体矩阵、尺寸、颜色映射 |
| layoutGrids | M07 布局与网格 | 提取栅格参数 |
| 图标组件 | M06 图标系统 | 统计图标尺寸、风格 |

### 步骤 2.5：颜色冲突检测

将归类后的色彩 Token 进行两两比对：

- **完全重复**（同色值不同 Token 名）→ 标注"确认是否为同色不同语义"
- **近似色**（ΔE ≤ 3）→ 建议统一
- **疑似冲突**（ΔE 3~5）→ 请用户确认

### 步骤 3：按模板生成文档

参照下方 13 个模块模板，将归类数据填入对应模块。**严格使用紧凑格式，禁止 Markdown 表格**（窄面板适配）。

---

## 输出模板规则

> **核心约束**：Figma Make 左侧面板较窄，**全部模块禁止使用 Markdown 表格**。每条信息换行展示，带字段标签前缀，层次分明。唯一例外是数据统计汇总处允许一处表格。

---

## 13 个模块模板

### M01 项目概述（必需）

```
# [产品名称] 设计规范

· 产品名称：[名称]
· 版本号：V x.x
· 更新日期：YYYY-MM-DD
· 设计工具：Figma
· Figma 文件：[链接]

## 版本记录

  · V1.0
    日期：YYYY-MM-DD
    内容：初始版本
    作者：[作者]

  · V1.1
    日期：YYYY-MM-DD
    内容：[变更内容]
    作者：[作者]
```

---

### M02 设计原则（必需）

```
# 设计原则

> 定义指导本产品所有设计决策的核心原则。

## 核心原则

  · [原则名]
    描述：[一句话描述]
    体现：[具体示例]

  · [原则名]
    描述：[一句话描述]
    体现：[具体示例]

## 设计决策记录

  · [决策主题]
    方案：[采用方案]，放弃 [备选方案]
    原因：[为什么]
```

---

### M03 色彩系统（必需）

```
# 色彩系统

## 品牌色

  · brand-primary
    色值：#XXXXXX
    用途：主按钮、关键链接
    Style：color/brand/primary

  · brand-secondary
    色值：#XXXXXX
    用途：辅助元素
    Style：color/brand/secondary

> AI 出图提示：主色调 #XXXXXX，视觉感受为 [温暖/冷静/活力/专业]

## 语义色

  · semantic-success
    色值：#XXXXXX
    用途：成功状态
    场景：[场景]

  · semantic-warning
    色值：#XXXXXX
    用途：警告状态
    场景：[场景]

  · semantic-error
    色值：#XXXXXX
    用途：错误/危险
    场景：[场景]

  · semantic-info
    色值：#XXXXXX
    用途：信息提示
    场景：[场景]

## 中性色

  · neutral-text-primary
    色值：#XXXXXX
    用途：主文字

  · neutral-text-secondary
    色值：#XXXXXX
    用途：辅助文字

  · neutral-text-disabled
    色值：#XXXXXX
    用途：禁用文字

  · neutral-border
    色值：#XXXXXX
    用途：边框/分割线

  · neutral-bg-page
    色值：#XXXXXX
    用途：页面背景

  · neutral-bg-container
    色值：#XXXXXX
    用途：卡片背景

  · neutral-bg-hover
    色值：#XXXXXX
    用途：悬停态背景

## 渐变色

  · [渐变名]
    色值：#XXXXXX → #XXXXXX
    角度：180°
    用途：[用途描述]

## 色彩使用规则
1. [规则1]
2. [规则2]
3. [规则3]
```

---

### M04 排版系统（必需）

```
# 排版系统

## 字体家族
· 中文正文：[字体名]，备选 [字体]，Style 前缀 font/cn
· 英文/数字：[字体名]，备选 [字体]，Style 前缀 font/en

## 字号梯度

  · display-xl
    规格：32px/40px · Bold
    场景：大标题

  · heading-lg
    规格：24px/32px · Bold
    场景：页面标题

  · heading-md
    规格：20px/28px · SemiBold
    场景：区域标题

  · heading-sm
    规格：16px/24px · SemiBold
    场景：模块标题

  · body-lg
    规格：16px/24px · Regular
    场景：正文大

  · body-md
    规格：14px/22px · Regular
    场景：正文标准

  · body-sm
    规格：12px/18px · Regular
    场景：辅助文字

  · caption
    规格：12px/16px · Regular
    场景：标签/注脚

## 排版规则
1. [规则]
2. [规则]
```

---

### M05 间距与圆角（推荐）

```
# 间距与圆角

## 间距梯度

  · space-xs
    值：4px
    场景：图标与文字间距

  · space-sm
    值：8px
    场景：紧密元素间距

  · space-md
    值：12px
    场景：标准元素间距

  · space-lg
    值：16px
    场景：模块间距

  · space-xl
    值：24px
    场景：区域间距

  · space-2xl
    值：32px
    场景：大区域间距

  · space-3xl
    值：48px
    场景：页面级间距

## 圆角规范

  · radius-none
    值：0px
    场景：全角元素

  · radius-sm
    值：4px
    场景：标签、小元素

  · radius-md
    值：8px
    场景：按钮、输入框

  · radius-lg
    值：12px
    场景：卡片、弹窗

  · radius-xl
    值：16px
    场景：大容器

  · radius-full
    值：9999px
    场景：头像、药丸形按钮

## 间距使用规则
1. 所有间距使用 4px 倍数体系
2. [规则]
```

---

### M06 图标系统（推荐）

```
# 图标系统

## 图标风格
· 风格：[线性/填充/双色]
· 线宽：[1.5px/2px]
· 圆角：[无圆角/圆角]

## 尺寸规范

  · icon-xs
    尺寸：12px
    场景：内联图标

  · icon-sm
    尺寸：16px
    场景：列表图标、按钮图标

  · icon-md
    尺寸：20px
    场景：导航图标

  · icon-lg
    尺寸：24px
    场景：空状态图标

  · icon-xl
    尺寸：32px
    场景：特性图标

## 图标使用规则
1. [规则]
2. [规则]
```

---

### M07 布局与网格（推荐）

```
# 布局与网格

## 栅格系统
· 列数：[12列/24列]
· 槽宽：[16px/24px]
· 边距：[24px]
· 最大宽度：[1200px]

## 断点规范

  · md
    宽度：≥1024px
    列数：[8]列
    场景：小屏桌面/折叠侧边栏

  · lg
    宽度：≥1280px
    列数：[12]列
    场景：标准桌面

  · xl
    宽度：≥1440px
    列数：[12]列
    场景：宽屏桌面

  · xxl
    宽度：≥1920px
    列数：[12]列
    场景：大屏桌面

## 容器规范

  · 页面容器
    宽度：最大1200px · 内边距24px
    场景：主内容区

  · 内容容器
    宽度：最大960px · 内边距24px
    场景：文章/表单

  · 紧凑容器
    宽度：最大720px · 内边距16px
    场景：对话框/侧边栏
```

---

### M08 组件规范（必需）

每个组件一个独立小节：

```
## [组件名称]

组件类型：[基础组件/复合组件]
Figma 路径：[Components/xxx]
使用频率：[高/中/低]

### 变体
  · Type：Primary / Secondary / Text / Danger
  · Size：Large / Medium / Small
  · State：Default / Hover / Active / Disabled

### 尺寸规范

  · Large
    高度：48px
    内边距：24-32px
    字号：16px
    圆角：8px

  · Medium
    高度：40px
    内边距：16-24px
    字号：14px
    圆角：8px

  · Small
    高度：32px
    内边距：8-16px
    字号：14px
    圆角：8px

### 间距规范
  · 水平排列间距：8px
  · 按钮与文字说明间距：4px

### 颜色规范

  · Primary
    Default：#XXX / Hover：#XXX / Active：#XXX / Disabled：#XXX

  · Danger
    Default：#XXX / Hover：#XXX / Active：#XXX / Disabled：#XXX

### 交互行为
  · Hover：背景色加深10%，0.2s 过渡
  · Click：背景色加深20%，按压缩放 0.98
  · Loading：文字替换为旋转图标，按钮禁用
  · Focus：显示 2px focus ring

### 使用规则
1. 一个页面最多出现 1 个 Primary 按钮
2. Danger 按钮仅用于不可逆操作，必须配合确认弹窗

### 禁止用法
  · 不要在表格每一行都放一个主按钮
  · 不要在弹窗中使用 Danger 按钮

> AI 出图提示：主按钮为圆角矩形，品牌色填充，白色文字，悬停时背景略加深
```

---

### M09 动效规范（可选）

```
# 动效规范

## 过渡时间

  · duration-fast
    时长：150ms
    场景：悬停、焦点

  · duration-normal
    时长：300ms
    场景：展开、收起

  · duration-slow
    时长：500ms
    场景：页面切换

## 缓动函数

  · ease-out
    曲线：cubic-bezier(0, 0, 0.2, 1)
    用途：进入动画

  · ease-in
    曲线：cubic-bezier(0.4, 0, 1, 1)
    用途：退出动画

  · ease-in-out
    曲线：cubic-bezier(0.4, 0, 0.2, 1)
    用途：循环动画

## 动效规则
1. 尊重用户"减少动效"系统设置
2. 功能性动画不超过 300ms
3. 装饰性动画不可干扰用户操作
```

---

### M10 图片与插图（可选）

```
# 图片与插图

## 图片规范
· 格式：WebP/PNG
· 最大文件大小：[200KB]
· 比例：[16:9]

## 插图风格
· 风格：[扁平/手绘/3D]
· 色彩：[与品牌色一致]
· 线条：[细线/无线]

> AI 出图提示：插画风格为扁平化设计，品牌色 #XXXXXX 为主色调，简洁线条，无阴影，适用于 B 端产品
```

---

### M11 暗色模式（可选）

```
# 暗色模式

## 映射规则

  · neutral-bg-page → dark-bg-page
    映射：#FFFFFF → #1A1A2E

  · neutral-text-primary → dark-text-primary
    映射：#333333 → #E5E5E5

## 暗色模式特殊规则
1. 品牌色降低饱和度 10%
2. 阴影用更深的背景色替代，不使用投影
3. 图片和插图降低亮度 10-15%
```

---

### M12 无障碍（可选）

```
# 无障碍规范

## 对比度要求
· 正文文字：WCAG AA ≥ 4.5:1，AAA ≥ 7:1
· 大号文字（≥18px Bold）：WCAG AA ≥ 3:1，AAA ≥ 4.5:1
· 图标/图形元素：≥ 3:1

## 可交互目标
· 可点击元素最小：24×24px，推荐：32×32px（触屏设备 44×44px）

## 无障碍规则
1. 所有图片添加 alt 文本
2. 不依赖颜色传递信息
3. 键盘可完全操作所有交互元素
```

---

### M13 术语表（可选）

```
# 术语表

  · [中文术语]
    英文：[English Term]
    定义：[简要定义]
    位置：[界面中出现的位置]
```

---

## 完整输出模板

```
# [产品名称] 设计规范

> · 文档生成日期：YYYY-MM-DD
> · Figma 文件：[链接]
> · 生成范围：[模块列表]
> · 数据来源：Figma 文件直接提取

---

## [M01] 项目概述

· 产品名称：[名称]
· 版本号：V x.x
· 更新日期：YYYY-MM-DD
· 设计工具：Figma

## 版本记录

  · V1.0
    日期：YYYY-MM-DD
    内容：初始版本
    作者：[作者]

---

## [M02] 设计原则

## 核心原则

  · [原则名]
    描述：[描述]
    体现：[示例]

---

## [M03] 色彩系统

## 品牌色

  · brand-primary
    色值：#XXXXXX
    用途：主按钮
    Style：color/brand/primary

## 语义色

  · semantic-success
    色值：#XXXXXX
    用途：成功
    场景：[场景]

## 中性色

  · neutral-text-primary
    色值：#XXXXXX
    用途：主文字

> AI 出图提示：主色调 #XXXXXX，[调性描述]

---

## [M04] 排版系统

## 字号梯度

  · heading-lg
    规格：24px/32px · Bold
    场景：页面标题

  · body-md
    规格：14px/22px · Regular
    场景：正文标准

---

## [M05] 间距与圆角

## 间距梯度

  · space-sm
    值：8px
    场景：紧密元素间距

  · space-md
    值：12px
    场景：标准元素间距

## 圆角规范

  · radius-md
    值：8px
    场景：按钮、输入框

---

[按选择的模块依次填充...]

---

## 附录

### 模块裁剪说明
[未生成的模块及原因]

### 颜色冲突记录

  · [类型]：Token A vs Token B
    色值：[色值A] / [色值B]
    ΔE：[N]
    处理：[处理结果]

### 待补充为 Variable 的颜色清单

  · #F5222D
    出现次数：58次
    建议：semantic/error
    类型：语义色

### 数据来源说明
[数据提取方式说明]

### 后续填充建议
[需要手动补充的内容]
```

---

## 模块选择快捷指令

| 用户指令 | 生成模块 |
|----------|----------|
| "完整规范" | M01-M05 + M07-M08（必需+推荐） |
| "精简规范" | M01-M04 + M08（仅必需） |
| "全部模块" | M01-M13 全部 |
| "速查版" | M03 + M04 + M08 核心数据 |
| "补充 [模块名]" | 在已有文档后追加指定模块 |

---

## 质量校验清单

生成后自检：

- [ ] 所有模块使用换行列表格式（每条独立一行带标签），无 Markdown 表格
- [ ] 色值统一为 `#XXXXXX` 格式
- [ ] 尺寸统一标注单位（px）
- [ ] 每个数据点可追溯到 Figma Style 名称或组件路径
- [ ] 推断数据标注 `[推断]`
- [ ] 必需模块已全部生成
- [ ] AI 出图提示嵌入在相关模块末尾
- [ ] 附录中的颜色冲突和 Variable 建议已输出
