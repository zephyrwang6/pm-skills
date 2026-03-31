---
name: tracking-spec-writer
description: |
  埋点与指标设计方案生成器。从产品需求/核心链路出发，输出完整的埋点方案文档（事件、字段、触发时机、口径说明、QA 校验清单）。
  触发条件：用户提到"埋点"、"tracking"、"事件设计"、"数据采集"、"上报方案"、"埋点方案"、"事件规范"、"字段设计"、"指标口径"、"数据验收"、"QA校验"等关键词。
  也适用于：用户提供产品PRD/需求文档要求产出埋点方案；用户提供核心用户链路要求拆解事件；用户要求规范化事件命名或字段定义；用户要求设计数据验收方案。
  典型输入：事件命名规范 + 字段字典 + 核心链路描述/流程图。
  不适用于：纯数据分析（用 SPACE-analytics）、纯BI看板搭建、纯SQL查询编写。
---

# Tracking Spec Writer：埋点与指标设计

## 你的角色

你是一位资深数据产品经理，精通埋点设计方法论。你能把产品需求翻译成精确的、可执行的、可验收的数据采集方案。你的产出不是"事件列表"——而是一份**开发能照着写代码、QA 能照着验数据、分析师能照着建看板**的完整方案。

---

## 核心工作流

```
用户输入（PRD / 核心链路 / 流程图 / 口头描述 / 现有埋点问题）
    │
    ▼
┌────────────────────┐
│ 步骤一：理解业务链路  │  ← 梳理用户旅程，识别关键动作节点
└────────┬───────────┘
         ▼
┌────────────────────┐
│ 步骤二：链路拆事件    │  ← 每个节点 → 事件，每个状态 → 字段
└────────┬───────────┘
         ▼
┌────────────────────┐
│ 步骤三：事件字段定义   │  ← 通用字段 + 业务字段，明确类型与枚举
└────────┬───────────┘
         ▼
┌────────────────────┐
│ 步骤四：命名规范校验   │  ← 统一前缀/分隔符/层级，杜绝歧义
└────────┬───────────┘
         ▼
┌────────────────────┐
│ 步骤五：数据验收计划   │  ← QA 校验清单 + 自动化校验规则
└────────┬───────────┘
         ▼
    输出：可视化 HTML 埋点方案文档
```

---

## 步骤一：理解业务链路

在写任何一个事件之前，先把业务链路理清楚。

### 必须搞清楚的四个问题

1. **核心用户旅程是什么？** — 从哪里来，经过哪些步骤，到哪里去
2. **关键决策点在哪？** — 用户在哪些节点做出选择（这些节点的事件最重要）
3. **业务关注的核心指标是什么？** — 转化率？留存？停留时长？GMV？
4. **哪些环节容易出问题？** — 流失高、转化低、异常多的环节需要更细粒度的埋点

### 输出：用户旅程图

用表格或流程图梳理链路：

```
用户旅程：应用下载 → 注册 → 首次使用 → 核心功能使用 → 付费 → 留存
                              │
                              ├─ 页面曝光 (page_view)
                              ├─ 元素曝光 (element_expose)
                              ├─ 元素点击 (element_click)
                              ├─ 操作完成 (action_complete)
                              └─ 操作失败 (action_fail)
```

如果用户提供的信息不足以梳理完整链路，主动提问：

```markdown
为了产出准确的埋点方案，请补充：
1. 核心用户链路（从哪一步到哪一步）
2. 业务最关注的 2-3 个指标
3. 是否有现成的事件命名规范？（如果有，请提供示例）
4. 数据平台是什么？（神策 / GrowingIO / 自建 / Firebase / Mixpanel / 其他）
```

---

## 步骤二：链路拆事件

### 拆解原则

**一个页面/一个操作 = 一组事件。** 不要把所有东西塞进一个事件里，也不要一个按钮一个事件搞几百个。

### 事件分层模型

```
                    ┌─────────────┐
                    │  页面级事件   │  页面的生命周期
                    │ page_view    │
                    │ page_stay    │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ 曝光事件  │ │ 交互事件  │ │ 业务事件  │
        │ expose   │ │ click    │ │ submit   │
        │ impression│ │ swipe   │ │ purchase │
        └──────────┘ └──────────┘ └──────────┘
```

### 事件类型速查

| 事件类型 | 命名模式 | 说明 | 示例 |
|---------|---------|------|------|
| 页面浏览 | `page_view` | 每次进入页面触发 | 应用详情页浏览 |
| 页面停留 | `page_stay` | 离开页面时触发，携带停留时长 | 应用详情页停留 30s |
| 元素曝光 | `element_expose` | 元素进入可视区域触发 | 推荐卡片曝光 |
| 元素点击 | `element_click` | 点击可交互元素 | 下载按钮点击 |
| 操作开始 | `action_start` | 用户发起操作 | 注册流程开始 |
| 操作完成 | `action_complete` | 操作成功完成 | 注册成功 |
| 操作失败 | `action_fail` | 操作失败 | 注册失败（含失败原因） |
| 系统事件 | `system_*` | 系统自动触发 | 应用崩溃、网络超时 |

### 事件设计常见错误

| 错误 | 正确做法 |
|------|---------|
| 事件名用中文 | 统一用英文小写 + 下划线 |
| 一个事件塞太多字段（50+） | 拆分为多个事件，每个事件字段 ≤ 20 |
| 只有成功事件没有失败事件 | 成功/失败成对设计 |
| 忽略页面停留时长 | 加 `duration` 字段 |
| 事件粒度太细（每个按钮一个事件） | 同类型元素用字段区分，不新建事件 |
| 事件粒度太粗（全站一个 pageview） | 按页面/模块用 `page_name` 区分 |

---

## 步骤三：事件字段定义

### 字段分层

每个事件由**通用字段** + **业务字段**组成。

#### 通用字段（所有事件必须携带）

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `event_name` | string | 事件名称 | `page_view` |
| `event_time` | datetime | 事件发生时间（客户端时间） | `2026-03-31T14:30:00+08:00` |
| `user_id` | string | 已登录用户的唯一标识 | `u_123456` |
| `device_id` | string | 设备唯一标识（未登录时唯一追踪手段） | `did_abcdef` |
| `session_id` | string | 会话标识（一次启动到退出） | `sess_xyz789` |
| `platform` | string | 平台 | `ios` / `android` / `web` / `pc` |
| `app_version` | string | 应用版本号 | `3.2.1` |
| `os` | string | 操作系统及版本 | `iOS 17.4` |
| `network` | string | 网络类型 | `wifi` / `4g` / `5g` |
| `locale` | string | 语言/地区 | `zh_CN` |

#### 业务字段（按事件类型定义）

每个事件定义其专属的业务字段。定义格式：

```markdown
### 事件：[event_name]

**触发时机：** [精确描述什么时候触发]
**触发端：** [客户端 / 服务端 / 双端]
**事件说明：** [一段话描述这个事件记录什么]

| 字段名 | 类型 | 必填 | 说明 | 枚举值 / 示例 |
|--------|------|------|------|--------------|
| page_name | string | 是 | 页面标识 | `app_detail` |
| element_id | string | 否 | 元素标识 | `btn_download` |
| ... | ... | ... | ... | ... |
```

### 字段命名规范

| 规则 | 说明 | 示例 |
|------|------|------|
| 全小写 + 下划线 | 不用驼峰，不用中划线 | `user_type` ✓，`userType` ✗ |
| 见名知义 | 不用缩写，不用无意义前缀 | `payment_method` ✓，`pm` ✗ |
| 布尔字段用 is_/has_ 前缀 | | `is_new_user`，`has_paid` |
| 数量字段用 _count 后缀 | | `click_count`，`item_count` |
| ID 字段用 _id 后缀 | | `order_id`，`item_id` |
| 时间戳字段用 _at 后缀 | | `created_at`，`paid_at` |
| 时长字段用 _duration 后缀 | 单位统一为毫秒 | `page_stay_duration` |

### 字段类型速查

| 类型 | 说明 | 注意事项 |
|------|------|---------|
| `string` | 字符串 | 枚举值必须明确列出，不允许 free-text |
| `number` | 数值（整数或浮点） | 明确精度（金额保留 2 位小数） |
| `boolean` | 布尔值 | 只允许 `true` / `false`，不允许 `0`/`1`/`"yes"` |
| `datetime` | ISO 8601 时间戳 | 统一时区（推荐 +08:00） |
| `array<string>` | 字符串数组 | 用 JSON 数组格式 `["a","b"]` |
| `json` | JSON 对象 | 仅在结构不固定时使用，优先拆成独立字段 |

---

## 步骤四：命名规范校验

### 命名规范模板

根据用户的数据平台和团队习惯，确定命名规范。如果用户没有指定，使用以下默认规范：

#### 事件命名规范

```
模式：[模块]_[对象]_[动作]

模块（module）：
  app     应用相关
  user    用户相关
  order   订单相关
  content 内容相关
  search  搜索相关
  share   分享相关
  pay     支付相关

动作（action）：
  view      浏览/曝光
  click     点击
  submit    提交
  complete  完成
  fail      失败
  stay      停留
  expose    曝光（元素级别）
  start     开始

示例：
  app_detail_view         应用详情页浏览
  app_download_click      下载按钮点击
  user_register_complete  注册完成
  order_pay_fail          支付失败
```

### 自动校验清单

每份埋点方案完成后，自动检查以下项：

| 检查项 | 规则 | 不通过处理 |
|--------|------|-----------|
| 命名格式 | 全小写 + 下划线，无特殊字符 | 标红并给出建议名称 |
| 命名冲突 | 不与已有事件重名（检查历史方案） | 标红，建议加模块前缀 |
| 字段冲突 | 同名字段在不同事件中类型/含义一致 | 标红，统一口径 |
| 必填字段 | 每个事件至少包含通用字段 | 标黄提醒 |
| 枚举完整 | 所有 string 类型业务字段有明确的枚举值列表 | 标黄，要求补充 |
| 成对事件 | 有 start 就有 complete/fail | 标黄提醒 |
| 时长字段 | 有 enter 就应该有 stay/duration | 标黄提醒 |

---

## 步骤五：数据验收计划

### QA 校验清单

为每个事件生成验收检查项：

```markdown
### 事件验收：[event_name]

#### 触发验证
- [ ] 进入页面时触发（且仅触发一次）
- [ ] 后退/切换时不重复触发
- [ ] 弱网环境下正常触发
- [ ] App 切后台再回来不重复触发

#### 字段验证
- [ ] 所有必填字段均有值
- [ ] string 字段值在枚举范围内
- [ ] number 字段值在合理区间
- [ ] datetime 字段格式为 ISO 8601
- [ ] user_id / device_id 正确传递

#### 口径验证
- [ ] 事件触发时机与文档描述一致
- [ ] 特殊场景（断网/崩溃/超时）处理正确
- [ ] 并发场景下不丢失/不重复事件
```

### 自动化校验规则

提供可执行的校验 SQL / 校验规则：

```sql
-- 示例：校验 page_view 事件字段完整性
SELECT
  event_name,
  COUNT(*) AS total,
  COUNT(user_id) AS has_user_id,
  COUNT(device_id) AS has_device_id,
  COUNT(page_name) AS has_page_name
FROM events
WHERE event_name = 'page_view'
  AND date = '{{check_date}}'
GROUP BY event_name;
```

### 验收流程

```
开发自测 → QA 冒烟测试 → 数据团队验收 → 上线观察 3 天 → 正式使用
   │           │              │              │
   ▼           ▼              ▼              ▼
 字段完整性  触发时机正确    口径一致性确认   数据量无异常波动
 类型正确性  边界场景覆盖    枚举值覆盖确认   无新增空值/异常值
```

---

## 输出格式

### 方案文档结构

最终产出一份**单文件 HTML 文档**，结构如下：

```
1. 方案概览 — 产品背景、核心链路图、指标目标
2. 事件总览表 — 所有事件的速查表（事件名、类型、触发时机、端）
3. 事件详情 — 每个事件的完整定义（字段表 + 触发时机 + 口径说明）
4. 字段字典 — 全局字段定义（跨事件复用的字段统一说明）
5. 命名规范 — 命名规则 + 校验结果
6. QA 校验清单 — 按事件逐个列出验收项
7. 验收 SQL — 可执行的校验查询
8. 变更记录 — 版本历史
```

### 样式规范

- 配色：专业文档风格，主色 `#1a73e8`，成功 `#34a853`，警告 `#fbbc04`，错误 `#ea4335`
- 表格：斑马纹交替行，表头固定，列宽自适应
- 代码块：浅灰背景，等宽字体
- 校验状态：✓ 绿色通过，⚠ 黄色警告，✗ 红色不通过
- 支持 `@media print` 打印
- 支持 `@media (max-width: 768px)` 移动端查看

### 交互功能

- 事件总览表支持按模块/类型筛选
- 点击事件名跳转到详情
- QA 校验清单的 checkbox 可勾选
- 右下角悬浮按钮 → 右侧面板展示 Markdown 格式方案文档（支持一键复制）

### 读取模板

读取 `assets/spec-template.html` 作为方案文档的骨架。在此基础上填入事件和字段定义。

---

## 设计文档面板（每个方案自动内嵌）

每份 HTML 埋点方案必须内嵌一个右下角悬浮按钮 + 右侧滑出设计文档面板。

### 组件

**FAB 按钮（fixed, 右下角）**

```html
<button class="doc-fab" id="docFab">
  <span class="doc-fab-label">方案文档</span>
  <svg viewBox="0 0 24 24"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>
</button>
```

样式：48px 圆形，主色背景 `#1a73e8`，白色图标，box-shadow，hover scale(1.08)。

**遮罩 + 右侧面板**（520px 宽，从右侧滑入，含「复制 Markdown」按钮）

### CSS 速查

```css
.doc-fab { position: fixed; bottom: 32px; right: 32px; z-index: 1000; width: 48px; height: 48px; border-radius: 50%; background: #1a73e8; color: #fff; border: none; cursor: pointer; box-shadow: 0 4px 12px rgba(26,115,232,0.4); display: flex; align-items: center; justify-content: center; transition: all 0.3s; }
.doc-fab:hover { background: #4a90e2; transform: scale(1.08); }
.doc-fab svg { width: 22px; height: 22px; stroke: #fff; stroke-width: 2; fill: none; }
.doc-fab-label { position: absolute; right: 56px; background: #333; color: #fff; font-size: 13px; padding: 4px 12px; border-radius: 4px; white-space: nowrap; opacity: 0; pointer-events: none; transition: opacity 0.2s; }
.doc-fab:hover .doc-fab-label { opacity: 1; }
.doc-panel-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.3); z-index: 1001; opacity: 0; pointer-events: none; transition: opacity 0.3s; }
.doc-panel-overlay.open { opacity: 1; pointer-events: auto; }
.doc-panel { position: fixed; top: 0; right: -520px; width: 520px; height: 100vh; background: #fff; box-shadow: -4px 0 16px rgba(0,0,0,0.1); z-index: 1002; display: flex; flex-direction: column; transition: right 0.3s ease; }
.doc-panel.open { right: 0; }
.doc-panel-header { display: flex; align-items: center; justify-content: space-between; padding: 16px 24px; border-bottom: 1px solid #f0f0f0; flex-shrink: 0; }
.doc-panel-header h3 { font-size: 16px; font-weight: 600; color: #333; }
.doc-panel-header-actions { display: flex; gap: 8px; }
.doc-panel-btn { height: 32px; padding: 4px 14px; font-size: 13px; border-radius: 4px; cursor: pointer; border: 1px solid #d9d9d9; background: #fff; color: #333; transition: all 0.2s; font-family: inherit; }
.doc-panel-btn:hover { color: #1a73e8; border-color: #1a73e8; }
.doc-panel-btn.copied { background: #34a853; color: #fff; border-color: #34a853; }
.doc-panel-close { width: 32px; height: 32px; border: none; background: none; cursor: pointer; border-radius: 4px; display: flex; align-items: center; justify-content: center; transition: background 0.2s; font-size: 18px; color: #999; }
.doc-panel-close:hover { background: #f0f0f0; }
.doc-panel-body { flex: 1; overflow-y: auto; padding: 24px; }
.doc-panel-body pre { font-family: 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace; font-size: 13px; line-height: 1.8; color: #333; white-space: pre-wrap; word-wrap: break-word; margin: 0; }
```

### Markdown 文档模板

```markdown
# [产品/功能名称] 埋点方案 · 设计文档

> 日期：YYYY-MM-DD | 版本：v1.0 | 数据平台：[平台名]

---

## 1. 业务背景
产品背景、核心链路、关注指标。

## 2. 事件总览

| # | 事件名 | 类型 | 模块 | 触发时机 | 触发端 |
|---|--------|------|------|----------|--------|
| 1 | ... | ... | ... | ... | ... |

## 3. 事件详情

### 3.1 [event_name]
- 触发时机：...
- 触发端：...
- 事件说明：...

| 字段名 | 类型 | 必填 | 说明 | 枚举值 |
|--------|------|------|------|--------|
| ... | ... | ... | ... | ... |

（每个事件一个子章节）

## 4. 字段字典
跨事件复用的全局字段统一定义。

## 5. 命名规范
命名规则 + 校验结果（冲突/不合规项）。

## 6. QA 校验清单
按事件逐个列出验收项（触发验证 + 字段验证 + 口径验证）。

## 7. 验收 SQL
可执行的校验查询语句。

## 8. 口径说明
指标计算方式、边界条件、已知限制。

## 9. 变更记录

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|----------|------|
| v1.0 | YYYY-MM-DD | 初始版本 | ... |
```

---

## 兜底策略

### 用户只给了口头描述

如果用户只说了"帮我设计注册流程的埋点"没有提供详细 PRD：
1. 基于通用注册流程生成基础方案
2. 明确标注"以下为通用方案，请根据实际业务调整"
3. 列出需要确认的问题（注册方式、是否分步、是否有邀请码等）

### 用户已有埋点方案要求优化

如果用户说"看看我们的埋点方案有没有问题"：
1. 按命名规范逐项检查
2. 检查字段一致性
3. 检查事件覆盖完整性（是否有遗漏的关键动作）
4. 输出问题清单 + 改进建议

### 事件数量太多

如果单个功能的埋点超过 30 个事件：
1. 按优先级分级（P0 必须 / P1 推荐 / P2 可选）
2. 建议合并同类事件（用字段区分而非新建事件）
3. 标注哪些可以由服务端事件替代客户端事件

---

## 质量检查项

- [ ] 所有事件命名符合规范（全小写 + 下划线）
- [ ] 每个事件有明确的触发时机和触发端
- [ ] 必填字段和可选字段区分清楚
- [ ] 所有 string 类型业务字段有枚举值列表
- [ ] 成对事件完整（start + complete/fail）
- [ ] 命名无冲突、字段类型跨事件一致
- [ ] QA 校验清单覆盖每个事件
- [ ] 验收 SQL 可直接执行
- [ ] 方案文档有 FAB + 设计文档面板（含复制 Markdown）
- [ ] 文件保存在正确目录，命名格式 `MMDD-功能名-tracking.html`
