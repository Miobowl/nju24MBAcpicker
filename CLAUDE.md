# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个MBA选课辅助工具，基于金币竞价系统帮助学生规划选课。采用单文件Web应用架构，零依赖，直接在浏览器中运行。

## 运行方式

直接在浏览器中打开 `index.html` 即可运行，无需构建或安装依赖。

## 技术栈

- 纯 HTML5 + CSS3 + Vanilla JavaScript
- 原生 Web APIs：Drag and Drop、LocalStorage、CSS Grid
- 响应式设计（断点：700px, 1000px, 1200px）

## 核心架构

**index.html** 包含全部代码（632行）：
- CSS 样式（三栏布局：课程列表 + 优先级栏 + 日历视图）
- HTML 结构
- JavaScript 逻辑

**关键数据结构**：
```javascript
const courses = [...]  // 71门课程数据（也可从courses.json加载）
let selections = { must: [], want: [], optional: [] }  // 用户选课
let coinAllocations = {}  // 金币分配
```

**核心算法**：
- `calculateDynamicCoins()` - 按优先级权重计算金币：必选×3, 想选×2, 可选×1
- `autoAllocateCoins()` - 智能分配1000枚总金币
- `suggestCoins()` - 建议金额并避开整数陷阱（50, 100, 150等）
- `getConflicts()` - 检测课程时间冲突

## 数据文件

- **courses.json** (2433行) - 主课程数据库，含71门课程的完整信息
- **course_data.json** (1841行) - 备用课程数据

课程数据结构：
```json
{
  "id": 1,
  "name": "课程名称",
  "sessions": [{ "date": "2026-03-05", "time": "9:00-12:00", "weekday": "四", "location": "907教室" }],
  "is_online": false,
  "credits": 1,
  "start_date": "2026-03-05",
  "end_date": "2026-03-06"
}
```

## 业务逻辑

1. 用户将课程拖拽到三个优先级栏（必选/想选/可选）
2. 系统按学分和优先级权重自动分配1000枚金币
3. 自动检测并警示时间冲突的课程
4. 日历视图展示3-8月的课程安排
5. 所有选择通过 localStorage 持久化
