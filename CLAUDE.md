# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个MBA选课辅助工具，基于金币竞价系统帮助学生规划选课。采用单文件Web应用架构，零依赖，直接在浏览器中运行。

**在线访问**: https://miobowl.github.io/nju24MBAcpicker/

## 运行方式

直接在浏览器中打开 `index.html` 即可运行，无需构建或安装依赖。

```bash
# 本地开发 - 直接打开或使用简单HTTP服务器
open index.html
# 或
python -m http.server 8000
```

## 技术栈

- 纯 HTML5 + CSS3 + Vanilla JavaScript
- 原生 Web APIs：Drag and Drop、LocalStorage、CSS Grid
- 响应式设计（断点：700px, 1100px, 1400px）
- 外部字体：Google Fonts（Noto Serif SC, Source Serif 4, ZCOOL XiaoWei）
- 设计风格：「墨韵学府」新文人商学风

## 文件结构

```
├── index.html          # 主应用（2126行，单文件包含HTML+CSS+JS）
├── courses.json        # 主课程数据库（2433行，71门课程）
├── course_data.json    # 备用课程数据（1841行）
├── README.md           # 用户文档
├── CLAUDE.md           # AI助手指南（本文件）
├── .gitignore          # Git忽略配置
└── docs/
    └── plans/          # 设计文档
        └── 2026-01-13-ux-enhancement-design.md  # UX增强设计规范
```

## 核心架构

### index.html 文件结构

```
行 1-1570:     CSS 样式（「墨韵学府」设计系统）
行 1571-1578:  HTML 结构（三栏布局 + 移动端导航 + Action Sheet）
行 1579-2123:  JavaScript 逻辑
```

### 关键数据结构

```javascript
// 课程数据（从嵌入的courses数组或courses.json加载）
const courses = [
  {
    id: 1,
    name: "课程名称",
    sessions: [{ date: "2026-03-05", time: "9:00-12:00", weekday: "四", location: "907教室" }],
    is_online: false,
    credits: 1,
    start_date: "2026-03-05",
    end_date: "2026-03-06"
  },
  ...
];

// 用户选课状态
let selections = {
  must: [],      // 必选课程ID数组
  want: [],      // 想选课程ID数组
  optional: []   // 可选课程ID数组
};

// 金币分配
let coinAllocations = {};  // { courseId: coinAmount }

// 优先级权重
const priorityWeights = { must: 3, want: 2, optional: 1 };

// 标签筛选状态
let activeFilters = new Set(['all']);

// 标签分类规则
const tagRules = {
  finance: ['投融资', '财富', '证券', '银行', '金融', ...],
  ai: ['AI', '人工智能', '区块链', '数字', '数据', ...],
  strategy: ['战略', '商业模式', '竞争', '并购', ...],
  culture: ['历史', '文化', '诸子', '葡萄酒', ...],
  marketing: ['品牌', '消费', '零售', '电商', ...],
  leadership: ['沟通', '团队', '人才', '领导', ...]
};
```

## 核心函数一览

### 金币算法
| 函数 | 说明 |
|------|------|
| `calculateDynamicCoins()` | 按优先级权重计算金币分配：必选×3, 想选×2, 可选×1 |
| `autoAllocateCoins()` | 执行智能分配并更新UI |
| `suggestCoins(priority, courseId)` | 计算建议金额，避开整数陷阱（50, 100, 150等） |

### 冲突检测
| 函数 | 说明 |
|------|------|
| `getConflicts(courseId)` | 返回与指定课程冲突的课程名称数组 |
| `getConflictIds(courseId)` | 返回冲突课程的ID数组 |
| `getConflictDates(courseId)` | 返回冲突的日期数组 |
| `highlightConflicts(courseId)` | 高亮显示冲突的课程卡片和日历日期 |
| `clearConflictHighlights()` | 清除所有冲突高亮 |

### 课程管理
| 函数 | 说明 |
|------|------|
| `getAllSelectedIds()` | 获取所有已选课程ID |
| `getCoursesPriority(id)` | 获取课程所在的优先级栏（must/want/optional/null） |
| `removeCourse(id, priority)` | 从指定优先级栏移除课程 |
| `updateCoin(id, value)` | 更新课程的金币分配 |

### 筛选系统
| 函数 | 说明 |
|------|------|
| `getCourseTag(course)` | 根据课程名称匹配标签分类 |
| `matchFilters(course)` | 检查课程是否匹配当前筛选条件 |

### 渲染函数
| 函数 | 说明 |
|------|------|
| `renderCourseList()` | 渲染左侧课程列表 |
| `renderPriorityLists()` | 渲染中间三个优先级栏 |
| `renderCalendar()` | 渲染右侧日历视图（3-8月） |
| `updateStats()` | 更新顶部统计信息 |
| `updateNavBadge()` | 更新移动端导航角标 |
| `updatePriorityHints()` | 更新优先级栏的空状态提示 |

### 拖拽系统
| 函数 | 说明 |
|------|------|
| `handleDragStart(e)` | 拖拽开始处理 |
| `handleDragEnd(e)` | 拖拽结束处理 |
| `setupDragAndDrop()` | 初始化拖拽监听器 |

### 移动端交互
| 函数 | 说明 |
|------|------|
| `isMobile()` | 检测是否为移动端（宽度≤700px） |
| `openActionSheet(courseId)` | 打开移动端课程选择弹窗 |
| `closeActionSheet()` | 关闭Action Sheet |
| `addToPriority(priority)` | 将课程添加到指定优先级栏 |
| `handleCourseCardClick(e)` | 处理移动端课程卡片点击 |

### 持久化
| 函数 | 说明 |
|------|------|
| `saveToLocalStorage()` | 保存选课和金币到localStorage |
| `loadFromLocalStorage()` | 从localStorage加载数据 |
| `init()` | 应用初始化入口 |

## 设计系统（墨韵学府）

### 色彩变量
```css
/* 核心色彩 - 墨韵靛青 */
--ink-900 ~ --ink-100   /* 灰度色阶 */

/* 宣纸白 - 背景 */
--paper, --paper-warm, --paper-cream

/* 暖金点缀 */
--gold, --gold-light, --gold-glow, --gold-border

/* 语义色彩 */
--vermilion     /* 朱砂红 - 必选/危险 */
--indigo        /* 靛青蓝 - 想选 */
--slate         /* 墨灰 - 可选 */
--jade          /* 翠绿 - 成功 */
--amber         /* 琥珀 - 警告 */
```

### 响应式断点
- `> 1400px`: 三栏布局（课程列表 + 优先级栏360px + 日历）
- `1100-1400px`: 三栏布局（优先级栏320px）
- `700-1100px`: 两栏布局
- `≤ 700px`: 单栏 + 底部Tab导航

## 业务逻辑

1. 用户将课程从左侧列表拖拽到中间的优先级栏（必选/想选/可选）
2. 系统按学分和优先级权重自动分配1000枚金币
3. 自动检测并警示时间冲突的课程（悬停高亮）
4. 日历视图展示3-8月的课程安排
5. 所有选择通过 localStorage 持久化（键：`mba_selections`, `mba_coins`）

### 金币分配算法

```
基础权重 = 课程学分 × 优先级权重
课程金币 = (基础权重 / 总权重) × 1000

// 避开整数陷阱
if (金币 % 50 === 0 || 金币 % 10 === 0) {
    金币 += 偏移量; // 使用 [3, -3, 7, -7, 13, -13, 17, -17] 循环
}
```

## 开发指南

### 修改时注意事项

1. **保持零依赖** - 不引入任何npm包或外部JS库
2. **单文件架构** - 所有CSS和JS都在index.html中
3. **保持向后兼容** - localStorage数据结构不可变更
4. **响应式优先** - 修改样式需测试三个断点
5. **移动端适配** - 拖拽功能在移动端改用Action Sheet

### 添加新课程分类标签

1. 在 `tagRules` 对象中添加新分类和关键词
2. 在HTML的 `filter-tags` div中添加对应按钮
3. 确保 `matchFilters()` 函数能正确处理新分类

### 测试清单

- [ ] 拖拽添加/移除课程
- [ ] 金币自动分配
- [ ] 冲突检测和高亮
- [ ] 日历视图正确显示
- [ ] localStorage持久化
- [ ] 移动端Action Sheet
- [ ] 三种屏幕尺寸布局

## 相关文档

- `docs/plans/2026-01-13-ux-enhancement-design.md` - UX增强设计规范，包含：
  - 冲突悬浮高亮交互设计
  - 课程标签筛选系统
  - 移动端底部Tab导航适配
