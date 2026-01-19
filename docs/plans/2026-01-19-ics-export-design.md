# ICS 日历导出功能设计

## 概述

为 MBA 选课助手添加 ICS 日历文件导出功能，方便用户将已选课程导入手机或电脑日历应用。

## 需求

- 导出范围：已选课程（必选 + 想选 + 可选）
- 事件内容：课程名称、上课时间、地点
- 文件结构：一个 ICS 文件包含多个 VEVENT 事件
- 课时处理：每节课单独一个事件
- 提醒设置：不设置，由用户自行配置

## 交互设计

### 触发方式

点击顶部「导出方案」按钮后，弹出下拉菜单：
- 📋 复制文本（原有功能）
- 📅 导出日历文件（新功能）

### 平台适配

- 桌面端（>700px）：按钮下方显示下拉菜单
- 移动端（≤700px）：复用 Action Sheet 组件，从底部弹出

### 空状态

未选择课程时，点击「导出日历文件」提示「请先选择课程」。

## ICS 事件格式

每个 VEVENT 包含：

| 字段 | 内容 | 示例 |
|------|------|------|
| SUMMARY | 课程名称 | `战略管理` |
| DTSTART | 上课开始时间 | `20260305T090000` |
| DTEND | 上课结束时间 | `20260305T120000` |
| LOCATION | 上课地点 | `907教室` |
| UID | 唯一标识 | `course-1-session-0@mba-picker` |

### 时间解析

课程数据 `time` 格式为 `"9:00-12:00"`，结合 `date` 字段解析为 ICS 时间戳格式 `YYYYMMDDTHHMMSS`。

### 线上课程

`is_online` 为 true 时，LOCATION 设为「线上课程」。

## UI 样式

复用「墨韵学府」设计系统：

```
下拉菜单：
- 背景：var(--paper-cream)
- 边框：var(--ink-200)
- 阴影：var(--shadow-lg)
- 悬停：var(--gold-glow)
```

菜单结构：
```
┌─────────────────────┐
│ 📋 复制文本         │
├─────────────────────┤
│ 📅 导出日历文件     │
└─────────────────────┘
```

## 实现方案

### 新增函数

| 函数 | 说明 |
|------|------|
| `generateICS()` | 生成完整 ICS 文件内容字符串 |
| `formatICSDate(date, time)` | 将日期和时间转换为 ICS 格式 |
| `downloadICS(content)` | 创建 Blob 并触发浏览器下载 |
| `toggleExportMenu()` | 切换导出菜单的显示/隐藏 |

### 修改现有代码

- 原「导出方案」按钮点击事件改为调用 `toggleExportMenu()`
- 原复制逻辑封装为 `copyPlanText()` 函数

### ICS 文件结构

```
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//MBA选课助手//CN
BEGIN:VEVENT
UID:course-1-session-0@mba-picker
DTSTART:20260305T090000
DTEND:20260305T120000
SUMMARY:战略管理
LOCATION:907教室
END:VEVENT
BEGIN:VEVENT
...更多事件...
END:VEVENT
END:VCALENDAR
```

## 文件命名

下载文件名：`MBA选课-YYYY-MM-DD.ics`（使用当前日期）
