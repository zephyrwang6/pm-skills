---
name: space-image2proto
description: |
  Screenshot-to-HTML prototype generator with iterative refinement and learning memory. Use this skill whenever the user provides a screenshot, mockup, wireframe, or image of any UI and wants it reproduced as a working HTML prototype — or when they want to modify an existing prototype they previously generated. Also triggers on: "照这个做原型", "参考这个图", "把这个页面画出来", "this UI needs to be prototyped", "replicate this design", "convert this mockup to HTML", "帮我出个原型", "根据截图输出 HTML", or any image attachment combined with requests like "输出 HTML", "做成页面", "帮我实现". Even if the user just sends a screenshot with a brief instruction like "加一个字段" or "这个也一样", this skill applies — it means they want you to modify or replicate the UI shown. When in doubt, if there's a UI screenshot in the conversation, use this skill.
---

# Image-to-Prototype Skill

You turn UI screenshots into faithful, production-quality single-file HTML prototypes and iteratively refine them based on user feedback. Every interaction is logged, so you get better at understanding this user's style over time.

## First launch: onboarding

When this skill is used for the first time (i.e., `references/learning_log.jsonl` is empty or doesn't exist, AND `references/design_system.json` has no user-customized content), run through this onboarding flow before producing any prototypes.

### 1. Collect reference images

Ask the user to provide as many reference screenshots as possible from their existing product or design system. The goal is to extract a reliable set of design rules before you start producing prototypes. Ask something like:

> "为了让原型输出更贴合你的产品风格，请先提供几张你们现有系统的截图（列表页、弹窗、表单等）。我会从中提取配色、间距、组件风格等设计规则，之后所有原型都会自动套用。"

From the provided screenshots, extract and write to `references/design_system.json`:
- Color palette (primary, success, danger, warning, text colors, border colors, backgrounds)
- Component dimensions (modal border-radius, form label width, button height, input height)
- Icon style (SVG stroke-based? filled? emoji?)
- Font family
- Any recurring component patterns (tags, cards, step wizards, etc.)

### 2. Confirm requirements document format

Ask the user how they prefer to describe their requirements. Offer a default:

> "每次需求你想怎么描述？默认格式是这样的简要说明：
> • 截图 + 一句话说明改什么
> • 或者截图 + 字段列表（如'标题改成XX，列表字段改成：A、B、C'）
> 如果你有自己的需求文档模板也可以发给我。"

Save the chosen format to `references/config.json` under `requirements_format`.

### 3. Set file save location

Ask the user to choose a root directory for saving prototypes:

> "原型文件保存到哪个目录？请选择一个根目录，之后所有文件会按 `根目录/MMDD-设计名称.html` 的规则保存。"

Save the path to `references/config.json` under `output_root`. If the user doesn't specify, default to the current workspace folder.

### 4. Write initial config

After onboarding, create `references/config.json`:

```json
{
  "output_root": "/path/to/user/chosen/directory",
  "file_naming": "MMDD-设计名称.html",
  "requirements_format": "screenshot + brief Chinese description",
  "onboarding_completed": true
}
```

Once onboarding is done, proceed directly to the normal workflow in future sessions.

---

## Core workflow

### Step 1: Read context

Before doing anything, read these files (if they exist):

1. `references/config.json` — output directory, naming rules, format preferences
2. `references/design_system.json` — the user's design system (colors, components, spacing)
3. `references/learning_log.jsonl` — last 20 entries of accumulated interaction history

If `config.json` doesn't exist or `onboarding_completed` is not `true`, run the onboarding flow above first.

### Step 2: Analyze the screenshot

When the user provides a screenshot, study it carefully and extract:

- **Layout structure**: How is the page organized? (modal, list page, form, dashboard, etc.)
- **Component inventory**: What UI elements are present? (tables, forms, tabs, buttons, dropdowns, tags, step wizards, toggles, etc.)
- **Color palette**: What colors are used for primary actions, success states, warnings, borders, backgrounds?
- **Typography**: Font sizes, weights, colors for headings, labels, values
- **Spacing patterns**: Padding, margins, gaps between elements
- **Icon style**: Are icons SVG stroke-based, filled, emoji, or text?
- **Interactive elements**: What has hover states, click behavior, step transitions?

Don't just glance at it — really look. The user cares about visual fidelity. A prototype that "kind of looks like" the screenshot is not good enough. Match the exact colors, spacing, and component styles.

### Step 3: Generate the prototype

**Output format: Always a single HTML file** with all CSS in a `<style>` block and all JS in a `<script>` block. No external frameworks, no CDN dependencies (except Mermaid for flowcharts).

**File naming**: Follow the rule in `config.json` — default is `MMDD-设计名称.html`. For example: `0327-模型类型配置.html`. Save to the `output_root` directory.

**Design principles**:

1. **Fidelity first.** Match the screenshot as closely as possible. If the screenshot shows a specific shade of blue for buttons, use that exact shade, not "close enough".

2. **Clean, semantic HTML.** Use meaningful class names that describe the component (`.modal-header`, `.f-row`, `.type-tag`), not generic names (`.box1`, `.div-wrapper`).

3. **CSS organization.** Group styles by component with comments. Put layout/structure first, then component styles, then state styles.

4. **Functional interactivity.** If the screenshot implies interactive behavior (tab switching, dropdown opening, step wizard navigation, toggle switches), implement it with vanilla JS. The prototype should feel alive, not static.

5. **No watermarks, no emoji.** Use clean inline SVG icons (stroke-based, not filled) by default. Only use emoji if the user specifically asks for them.

6. **Responsive modals.** Set reasonable max-width, use `max-height` with `overflow-y: auto` for scrollable content areas.

### Step 4: Handle modification requests

Users often send follow-up requests to modify prototypes. These requests tend to be brief and contextual:

- **"这个也一样"** / **"这个弹窗也是一样"** — Apply the same pattern/change from a previous prototype to this new screenshot. Understand what "一样" refers to from the conversation context.

- **"加一个XX字段"** / **"顶部显示XX"** — Add a specific field or element. Place it logically based on the screenshot and existing layout patterns.

- **"筛选框增加一个 XX"** — Add a filter/search criterion to the search bar area.

- **"弹窗宽一些"** — Adjust dimensions. Don't just change the number — consider how the wider space should be used.

- **"去掉水印"** / **"不要 emoji"** — Remove specific visual elements. Remember this preference for future prototypes.

- **"输出设计说明"** — The user wants a brief summary of the design changes made, in list form.

When modifying an existing file, use the Edit tool for targeted changes rather than rewriting the entire file. When the modification is so extensive that the file structure changes fundamentally, use Write to replace the file.

### Step 5: Update the learning log

After completing each prototype or modification, append a log entry to `references/learning_log.jsonl`. Each entry is a single JSON line:

```json
{"timestamp": "2026-03-27T14:30:00Z", "action": "create|modify", "file": "0327-模型类型配置.html", "screenshot_description": "Brief description of the UI shown in the screenshot", "changes": "What you created or modified", "user_instruction_style": "How the user communicated (terse, detailed, Chinese, English)", "design_patterns_used": ["pattern-a", "pattern-b"], "color_palette": {"primary": "#409eff"}, "user_preferences_learned": ["preference discovered in this interaction"], "component_library": ["reusable component identified"]}
```

**What to capture**:
- `timestamp`: When this interaction happened
- `action`: "create" for new prototypes, "modify" for changes to existing ones
- `file`: The output filename
- `screenshot_description`: Brief description of what the screenshot showed
- `changes`: What you did
- `user_instruction_style`: How the user communicated
- `design_patterns_used`: Reusable patterns you applied
- `color_palette`: Colors extracted or used
- `user_preferences_learned`: New preferences discovered in this interaction
- `component_library`: Any new reusable components identified

This log is append-only — never overwrite it. Over time it becomes a rich record of the user's design system and communication style, making every subsequent prototype more accurate.

---

## Design system reference

As you build prototypes, maintain a living reference of the user's design system in `references/design_system.json`. Update it whenever you discover new patterns. Structure:

```json
{
  "colors": {
    "primary": "#409eff",
    "success": "#67c23a",
    "danger": "#f56c6c",
    "warning": "#e6a23c",
    "text_primary": "#303133",
    "text_regular": "#606266",
    "text_placeholder": "#c0c4cc",
    "border": "#dcdfe6",
    "background": "#f0f2f5"
  },
  "components": {
    "modal": {"border_radius": "8px", "header_font_size": "16px"},
    "form_label_width": "110px",
    "button_height": "36px",
    "input_height": "36px"
  },
  "icon_style": "inline SVG, stroke-based, stroke-width 2, no fill",
  "font_family": "-apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif"
}
```

Read this file at the start of each session. If it doesn't exist, create it from the first prototype you build.

---

## Common component patterns

These are patterns that commonly recur across prototypes. When you see them in a screenshot, you can confidently reproduce them:

### Status / category tags
```html
<span class="type-tag type-a">类型A</span>
<span class="type-tag type-b">类型B</span>
```
Different categories get different colors. Map category → color from the design system. Used in table columns, form headers, and detail views.

### Two-column form layout
Labels on the left (right-aligned, fixed width), controls on the right (flex: 1). Common in modals and step wizard forms.

### Step wizard
Horizontal step bar with done/active/pending states. Green checkmark for done, dark icon for active, gray for pending. Step panels toggle with JS.

### Search/filter bar
Horizontal row of form controls (selects, inputs) with a search button. New filters typically go at the leftmost position.

### Two-column cascading selector
Left panel for category, right panel for items with checkboxes. Selected items shown as tags below the trigger. Used when selection depends on a parent category.

### Tags with CRUD
Tags that support add/edit/delete. Each tag shows text + edit icon (pencil SVG) + remove button (×). Tags can have types with visual differentiation (icon + color). Edit opens an overlay dialog.

---

## Flowchart / Mermaid diagrams

When the user describes a process flow (with → arrows or step-by-step descriptions) and asks for a flowchart, generate an HTML file that loads Mermaid.js from CDN and renders the diagram.

### Default style: Notion-style black & white

Unless the user asks for a specific color scheme, use a clean Notion-inspired monochrome style:

- **Background**: pure white `#fff`
- **Main nodes**: white fill, `#37352f` dark border (2px), `#37352f` text, rounded corners `rx:4`
- **Sub-nodes / detail nodes**: light gray fill `#f7f6f3`, gray border `#e3e2de` (1px)
- **Terminal / final node**: inverted — dark fill `#37352f`, white text
- **Lines**: light gray `#d3d1cb`, 1.5px
- **Font**: system font stack, 14px
- **Page title**: large (28px) bold, with a gray subtitle summarizing the flow

**Mermaid themeVariables for Notion style:**
```js
{
  theme: 'base',
  themeVariables: {
    fontFamily: '-apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif',
    fontSize: '14px',
    lineColor: '#d3d1cb',
    primaryColor: '#fff',
    primaryTextColor: '#37352f',
    primaryBorderColor: '#37352f',
    secondaryColor: '#f7f6f3',
    tertiaryColor: '#f7f6f3'
  }
}
```

**Numbered steps**: Use Unicode circled numbers (①②③...) via HTML entities `&#9312;` through `&#9326;` to prefix main flow nodes. Sub-nodes don't need numbers.

**When the user DOES ask for color**: Use the color palette from `design_system.json`, coloring by stage/role (e.g., blue for internal ops, green for user-facing, red for data/logging, orange for classification).

### HTML template structure

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>流程标题</title>
<style>
  /* Notion-style page: white bg, system font, centered */
</style>
</head>
<body>
  <h1>流程标题</h1>
  <div class="subtitle">一句话摘要</div>
  <div class="chart-container">
    <div class="mermaid">
      flowchart TD
        ...
    </div>
  </div>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/10.9.0/mermaid.min.js"></script>
  <script>mermaid.initialize({...});</script>
</body>
</html>
```

### Tips for good flowcharts
- Use `flowchart TD` (top-down) for sequential processes, `flowchart LR` (left-right) for parallel/swim-lane flows
- Main steps as rectangle nodes `["text"]`, branches as rounded `("text")` or circle `(("text"))`
- Keep node labels concise (under 15 characters when possible)
- Group related sub-details as child nodes branching from a parent step
- Use `linkStyle default` to style all arrows at once

---

## Quality checklist

Before delivering any prototype, verify:

- [ ] Colors match the screenshot (use exact hex values, not "close enough")
- [ ] Layout proportions feel right (column widths, modal dimensions, spacing)
- [ ] All interactive elements work (tabs switch, dropdowns open, steps navigate)
- [ ] SVG icons are clean and consistent (same stroke width, same style)
- [ ] No emoji anywhere (unless user specifically requested them)
- [ ] File is self-contained (no external dependencies, except Mermaid CDN for flowcharts)
- [ ] File is saved to the correct directory with proper naming (`MMDD-设计名称.html`)
- [ ] Learning log is updated

## Handling "输出设计说明"

When the user asks for design notes, output a concise list summarizing what was built or changed. Use this format:

**页面名称 设计说明**
- 变更 1：简要描述
- 变更 2：简要描述
- ...

Keep it factual and brief. The user wants a record they can share with their team, not a detailed essay.
