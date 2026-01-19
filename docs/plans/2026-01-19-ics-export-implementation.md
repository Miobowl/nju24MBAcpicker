# ICS 日历导出功能实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 为 MBA 选课助手添加 ICS 日历文件导出功能，将已选课程导出为标准日历文件。

**Architecture:** 在现有「导出方案」按钮上添加下拉菜单，提供「复制文本」和「导出日历文件」两个选项。桌面端使用下拉菜单，移动端复用 Action Sheet 组件。ICS 生成逻辑为纯 JavaScript 字符串拼接。

**Tech Stack:** Vanilla JavaScript, HTML5 Blob API, 无外部依赖

---

## Task 1: 添加导出菜单 CSS 样式

**Files:**
- Modify: `index.html:203-208`（在 `.export-btn:disabled` 样式后添加）

**Step 1: 添加下拉菜单样式**

在 `index.html` 第 208 行（`.export-btn:disabled` 结束大括号后）添加：

```css
/* 导出菜单容器 */
.export-menu-wrapper {
    position: relative;
    display: inline-block;
}

/* 导出下拉菜单 */
.export-menu {
    display: none;
    position: absolute;
    top: calc(100% + 8px);
    right: 0;
    min-width: 180px;
    background: var(--paper-cream);
    border: 1px solid var(--ink-200);
    border-radius: 8px;
    box-shadow: var(--shadow-lg);
    z-index: 100;
    overflow: hidden;
    animation: menuFadeIn 0.15s ease-out;
}

.export-menu.active {
    display: block;
}

@keyframes menuFadeIn {
    from { opacity: 0; transform: translateY(-8px); }
    to { opacity: 1; transform: translateY(0); }
}

.export-menu-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px 16px;
    background: none;
    border: none;
    width: 100%;
    text-align: left;
    font-family: 'Noto Serif SC', serif;
    font-size: 0.9rem;
    color: var(--ink-700);
    cursor: pointer;
    transition: var(--transition-fast);
}

.export-menu-item:hover {
    background: var(--gold-glow);
    color: var(--ink-900);
}

.export-menu-item:not(:last-child) {
    border-bottom: 1px solid var(--ink-100);
}

.export-menu-item .icon {
    font-size: 1.1rem;
}
```

**Step 2: 验证**

在浏览器中打开 `index.html`，使用开发者工具检查样式是否正确加载（无 CSS 语法错误）。

**Step 3: Commit**

```bash
git add index.html
git commit -m "style: 添加导出菜单下拉样式"
```

---

## Task 2: 修改 HTML 结构

**Files:**
- Modify: `index.html:1549`（导出按钮）

**Step 1: 用菜单容器包裹导出按钮**

将第 1549 行：
```html
<button class="export-btn" id="exportBtn" onclick="exportSelections()">导出方案</button>
```

替换为：
```html
<div class="export-menu-wrapper">
    <button class="export-btn" id="exportBtn" onclick="toggleExportMenu(event)">导出方案</button>
    <div class="export-menu" id="exportMenu">
        <button class="export-menu-item" onclick="copyPlanText()">
            <span class="icon">📋</span>
            <span>复制文本</span>
        </button>
        <button class="export-menu-item" onclick="exportToICS()">
            <span class="icon">📅</span>
            <span>导出日历文件</span>
        </button>
    </div>
</div>
```

**Step 2: 验证**

在浏览器中刷新页面，检查按钮仍正常显示，下拉菜单默认隐藏。

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 添加导出菜单 HTML 结构"
```

---

## Task 3: 实现菜单切换函数

**Files:**
- Modify: `index.html`（在 `exportSelections` 函数前添加，约第 2267 行）

**Step 1: 添加 toggleExportMenu 函数**

在 `// 导出选课方案` 注释前添加：

```javascript
// 导出菜单控制
function toggleExportMenu(event) {
    event.stopPropagation();
    const menu = document.getElementById('exportMenu');
    menu.classList.toggle('active');
}

// 点击外部关闭菜单
document.addEventListener('click', function(e) {
    const menu = document.getElementById('exportMenu');
    const wrapper = e.target.closest('.export-menu-wrapper');
    if (!wrapper && menu) {
        menu.classList.remove('active');
    }
});
```

**Step 2: 验证**

1. 刷新页面
2. 点击「导出方案」按钮，菜单应弹出
3. 再次点击按钮，菜单应收起
4. 点击页面其他区域，菜单应收起

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 实现导出菜单切换逻辑"
```

---

## Task 4: 重构原有导出函数

**Files:**
- Modify: `index.html:2268-2355`（重命名 `exportSelections` 为 `copyPlanText`）

**Step 1: 重命名函数并添加关闭菜单逻辑**

将函数名从 `exportSelections` 改为 `copyPlanText`，并在函数开头添加关闭菜单：

```javascript
// 导出选课方案（复制文本）
function copyPlanText() {
    // 关闭菜单
    document.getElementById('exportMenu').classList.remove('active');

    const allSelected = getAllSelectedIds();
    // ... 保持原有逻辑不变
}
```

**Step 2: 验证**

1. 刷新页面，选择几门课程
2. 点击「导出方案」→「复制文本」
3. 粘贴验证内容正确

**Step 3: Commit**

```bash
git add index.html
git commit -m "refactor: 重命名 exportSelections 为 copyPlanText"
```

---

## Task 5: 实现 ICS 时间格式化函数

**Files:**
- Modify: `index.html`（在 `copyPlanText` 函数后添加）

**Step 1: 添加 formatICSDate 函数**

```javascript
// ICS 日期格式化
// 输入: date="2026-03-05", time="9:00-12:00"
// 输出: { start: "20260305T090000", end: "20260305T120000" }
function formatICSDate(date, time) {
    // 移除日期中的横线
    const dateStr = date.replace(/-/g, '');

    // 解析时间 "9:00-12:00" 或 "09:00-12:00"
    const [startTime, endTime] = time.split('-');

    function formatTime(t) {
        const [hour, minute] = t.split(':');
        return hour.padStart(2, '0') + minute.padStart(2, '0') + '00';
    }

    return {
        start: dateStr + 'T' + formatTime(startTime),
        end: dateStr + 'T' + formatTime(endTime)
    };
}
```

**Step 2: 验证**

在浏览器控制台测试：
```javascript
formatICSDate("2026-03-05", "9:00-12:00")
// 期望: { start: "20260305T090000", end: "20260305T120000" }

formatICSDate("2026-05-16", "14:00-17:00")
// 期望: { start: "20260516T140000", end: "20260516T170000" }
```

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 添加 ICS 日期格式化函数"
```

---

## Task 6: 实现 ICS 内容生成函数

**Files:**
- Modify: `index.html`（在 `formatICSDate` 函数后添加）

**Step 1: 添加 generateICS 函数**

```javascript
// 生成 ICS 文件内容
function generateICS() {
    const allSelected = getAllSelectedIds();
    if (allSelected.length === 0) {
        return null;
    }

    let ics = [
        'BEGIN:VCALENDAR',
        'VERSION:2.0',
        'PRODID:-//MBA选课助手//CN',
        'CALSCALE:GREGORIAN',
        'METHOD:PUBLISH'
    ];

    allSelected.forEach(id => {
        const course = courses.find(c => c.id === id);
        if (!course || !course.sessions) return;

        course.sessions.forEach((session, idx) => {
            const times = formatICSDate(session.date, session.time);
            const location = course.is_online ? '线上课程' : (session.location || '待定');
            const uid = `course-${id}-session-${idx}@mba-picker`;

            ics.push('BEGIN:VEVENT');
            ics.push(`UID:${uid}`);
            ics.push(`DTSTART:${times.start}`);
            ics.push(`DTEND:${times.end}`);
            ics.push(`SUMMARY:${course.name}`);
            ics.push(`LOCATION:${location}`);
            ics.push('END:VEVENT');
        });
    });

    ics.push('END:VCALENDAR');

    return ics.join('\r\n');
}
```

**Step 2: 验证**

在浏览器中选择几门课程，然后在控制台执行：
```javascript
console.log(generateICS())
```

检查输出的 ICS 格式是否正确。

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 添加 ICS 内容生成函数"
```

---

## Task 7: 实现 ICS 下载函数

**Files:**
- Modify: `index.html`（在 `generateICS` 函数后添加）

**Step 1: 添加 downloadICS 和 exportToICS 函数**

```javascript
// 下载 ICS 文件
function downloadICS(content, filename) {
    const blob = new Blob([content], { type: 'text/calendar;charset=utf-8' });
    const url = URL.createObjectURL(blob);

    const link = document.createElement('a');
    link.href = url;
    link.download = filename;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    URL.revokeObjectURL(url);
}

// 导出为 ICS 日历文件
function exportToICS() {
    // 关闭菜单
    document.getElementById('exportMenu').classList.remove('active');

    const allSelected = getAllSelectedIds();
    if (allSelected.length === 0) {
        showToast('请先选择课程', 'error');
        return;
    }

    const icsContent = generateICS();
    if (!icsContent) {
        showToast('生成日历文件失败', 'error');
        return;
    }

    // 生成文件名：MBA选课-2026-01-19.ics
    const today = new Date();
    const dateStr = today.toISOString().split('T')[0];
    const filename = `MBA选课-${dateStr}.ics`;

    downloadICS(icsContent, filename);
    showToast('日历文件已下载', 'success');
}
```

**Step 2: 验证**

1. 刷新页面，选择几门课程
2. 点击「导出方案」→「导出日历文件」
3. 检查是否下载了 `.ics` 文件
4. 用文本编辑器打开验证格式
5. 用日历应用（如 macOS 日历、Outlook）导入验证

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 实现 ICS 文件下载功能"
```

---

## Task 8: 移动端适配

**Files:**
- Modify: `index.html`（修改 `toggleExportMenu` 函数）

**Step 1: 检测移动端并使用 Action Sheet**

修改 `toggleExportMenu` 函数：

```javascript
// 导出菜单控制
function toggleExportMenu(event) {
    event.stopPropagation();

    // 移动端使用 Action Sheet
    if (isMobile()) {
        openExportActionSheet();
        return;
    }

    // 桌面端使用下拉菜单
    const menu = document.getElementById('exportMenu');
    menu.classList.toggle('active');
}

// 打开导出 Action Sheet（移动端）
function openExportActionSheet() {
    const actionSheet = document.getElementById('actionSheet');
    const title = document.getElementById('actionSheetTitle');
    const courseInfo = document.getElementById('actionSheetCourse');
    const options = document.querySelector('.action-sheet-options');

    title.textContent = '导出方案';
    courseInfo.style.display = 'none';

    options.innerHTML = `
        <button class="action-option" onclick="copyPlanText(); closeActionSheet();">
            <span class="option-icon" style="background: var(--indigo-bg); color: var(--indigo);">📋</span>
            <div class="option-text">
                <div class="option-title">复制文本</div>
                <div class="option-desc">复制选课方案到剪贴板</div>
            </div>
        </button>
        <button class="action-option" onclick="exportToICS(); closeActionSheet();">
            <span class="option-icon" style="background: var(--jade-bg); color: var(--jade);">📅</span>
            <div class="option-text">
                <div class="option-title">导出日历文件</div>
                <div class="option-desc">下载 ICS 日历文件</div>
            </div>
        </button>
    `;

    actionSheet.classList.add('active');
}
```

**Step 2: 验证**

1. 使用浏览器开发者工具切换到移动端视图（≤700px）
2. 点击「导出方案」按钮
3. 应从底部弹出 Action Sheet
4. 点击两个选项分别验证功能

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 移动端导出使用 Action Sheet"
```

---

## Task 9: 最终测试与清理

**Files:**
- Review: `index.html`

**Step 1: 完整功能测试**

| 场景 | 操作 | 预期结果 |
|------|------|----------|
| 桌面端复制文本 | 选课 → 导出方案 → 复制文本 | 显示成功提示，剪贴板有内容 |
| 桌面端导出日历 | 选课 → 导出方案 → 导出日历文件 | 下载 .ics 文件 |
| 移动端复制文本 | 选课 → 导出方案 → 复制文本 | Action Sheet 弹出，复制成功 |
| 移动端导出日历 | 选课 → 导出方案 → 导出日历文件 | Action Sheet 弹出，下载文件 |
| 无选课时导出 | 不选课 → 导出日历文件 | 显示「请先选择课程」提示 |
| ICS 格式验证 | 导入到日历应用 | 事件正确显示时间、地点 |

**Step 2: 删除测试时可能添加的 console.log**

搜索并删除调试代码。

**Step 3: Final Commit**

```bash
git add index.html
git commit -m "test: 完成 ICS 导出功能测试"
```

---

## 验收标准

1. 点击「导出方案」按钮弹出下拉菜单（桌面端）或 Action Sheet（移动端）
2. 「复制文本」功能与原有行为一致
3. 「导出日历文件」生成有效的 ICS 文件
4. ICS 文件可被主流日历应用（Apple 日历、Google 日历、Outlook）正确导入
5. 每门课程的每节课时显示为独立事件
6. 未选课时给出友好提示
