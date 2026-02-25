本文档定义**课表解析脚本**的接口规范：脚本类型、执行环境、传入参数、可用变量、返回值格式。

本平台提供三类脚本（HTML 解析、Excel 解析、JS 注入），返回值字段与彼处略有不同（如使用 `courseName`、`room`、`day`、`startNode`、`step`、`startWeek`、`endWeek`、`type`），请以本文档约定为准。

---

## 一、脚本类型与执行环境

| 脚本类型 | 执行环境 | 触发时机 |
|----------|----------|----------|
| **HTML 课表解析** | 服务端，收到整页 HTML 字符串 | 用户在 App 内从教务系统页点击「提取课表」后，系统抓取 HTML 并调用您的脚本 |
| **Excel 课表解析** | 服务端，收到 Excel 文件内容 | 用户选择 Excel 导入并上传文件后，系统调用您的脚本 |
| **JS 注入** | 用户设备上打开的课表页面（浏览器/WebView） | 用户点击「提取课表」后，系统在课表页内注入并执行您的脚本 |

---

## 二、HTML 课表解析脚本规范

### 2.1 执行方式

脚本在服务端以**同步**方式执行，形式等价于：

```js
(function(html) {
  // 您的脚本代码（即「解析代码」框中的全部内容）
  return result;
})(html);
```

### 2.2 传入参数与可用变量

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `html` | string | 当前课表页面的完整 HTML（含系统合并的 iframe 内容） |
| `JSON` | object | 标准 JSON 对象，可用于解析/序列化 |

### 2.3 返回值规范

脚本**必须**返回一个对象，且包含下表字段；否则将按解析失败处理，系统会回退到通用解析。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `scheduleConfig` | object | 是 | 课表配置，见下表 |
| `courses` | array | 是 | 课程安排列表，每项格式见「单条课程」表 |
| `timeNodes` | array | 否 | 节次时间表，见下表 |

**scheduleConfig 结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `semester` | string | 学期，如 `2024-2025-1` |
| `startDate` | string | 学期开始日期，如 `2024-09-01`（可选） |
| `maxWeek` | number | 最大周数，如 20 |
| `tableName` | string | 课表名称，如「我的课表」 |
| `timeNodes` | array | 也可放在此处，与顶层 `timeNodes` 二选一 |

**timeNodes 单项结构（可选）：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `node` | number | 节次编号，从 1 开始 |
| `startTime` | string | 上课时间，如 `08:00` |
| `endTime` | string | 下课时间，如 `08:45` |

**单条课程（courses 数组元素）结构：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `courseName` | string | 是 | 课程名称 |
| `teacher` | string | 否 | 教师 |
| `room` | string | 否 | 教室/地点 |
| `day` | number | 是 | 星期几，1–7（1 为周一） |
| `startNode` | number | 是 | 开始节次，从 1 开始 |
| `step` | number | 否 | 连续节数，默认 2 |
| `startWeek` | number | 是 | 开始周 |
| `endWeek` | number | 是 | 结束周 |
| `type` | number | 否 | 0 每周 / 1 单周 / 2 双周，默认 0 |

### 2.4 示例（逻辑说明）

- 从 `html` 中用正则或字符串匹配定位课表表格、学期等。
- 仅解析数据行（如带特定 class 的 `<tr>`），避免把表头当数据。
- 同一门课、同一天、连续节次建议合并为一条，`step` 设为连续节数。
- 在脚本字符串中写正则时，反斜杠需双写（如 `\\d`、`\\s`），以免执行时报错。

---

## 三、Excel 课表解析脚本规范

### 3.1 执行方式

脚本在服务端以**同步**方式执行，形式等价于：

```js
(function(buffer, semester) {
  // 您的脚本代码；可使用 getExcelRows(buffer) 获取二维数组
  return result;
})(buffer, semester);
```

### 3.2 传入参数与可用变量

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `buffer` | Buffer | Excel 文件二进制内容 |
| `semester` | string | 用户选择的学期（若有） |
| `getExcelRows(buffer)` | function | 调用后返回二维数组 `[row][col]`，即表格行→列 |
| `JSON` | object | 标准 JSON 对象 |

### 3.3 返回值规范

脚本**必须**返回一个对象，且包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `scheduleConfig` | object | 同「二、2.3」中 scheduleConfig 结构 |
| `courseList` | array | 课程列表，每项含 `id`（自增）、`courseName`、`teacher`、`credit`（可选）、`note`（可选）等 |
| `courseSchedules` | array | 排课列表，每项含 `id`（对应 courseList 的 id）、`day`、`startNode`、`step`、`room`、`startWeek`、`endWeek`、`type` |

`courseList` 与 `courseSchedules` 通过 `id` 一一对应，`id` 从 1 自增即可。

---

## 四、JS 注入脚本规范

### 4.1 执行环境

脚本在用户 App 内打开的**课表页面**中执行（浏览器/WebView 环境），可访问当前页的 `document`、`window`、Cookie、`fetch` 等，也可在 iframe 内执行（系统会尽量注入到含课表内容的同源 iframe）。

### 4.2 执行方式与返回值

- 脚本会被注入到页面并执行，**执行结果**通过约定格式回传给 App。
- 返回值可为：
  - **同步**：直接返回对象 `{ scheduleConfig, courses [, timeNodes ] }`。
  - **异步**：返回 `Promise<{ scheduleConfig, courses [, timeNodes ] }>`。
- **courses** 与 **scheduleConfig / timeNodes** 的字段规范与「二、2.3」一致。

### 4.3 必须遵守的约定

1. **脚本入口**
   - 整段注入代码**必须以 `return` 开头**，例如：`return (function () { ... })();`
   - 否则运行环境无法拿到返回值，会判定为无数据并可能回退到抓取 HTML 解析。

2. **字符串与正则**
   - 注入前脚本会经过 JSON 序列化，正则中的反斜杠可能导致执行时报错（如 Invalid escape）。
   - 建议：正则中避免 `\d`、`\s`、`\/`，改用 `[0-9]`、空格、`[/]` 等；空字符串用双引号 `""`。若必须使用反斜杠，请写成 `\\`。

3. **连续节次**
   - 若数据按「单节」返回，请在脚本内将「同一门课、同一天、节次连续」的条目合并为一条，并设置 `step` 为连续节数；合并所用键（课程名、教室、周次等）须与最终输出字段一致。

4. **iframe**
   - 课表若在 iframe 中，脚本需自行定位到该 iframe 的 `contentWindow` / `document` 再发请求或读 DOM。

5. **超时**
   - App 约 18 秒内未收到结果将回退为抓取 HTML。建议脚本内请求超时设为 3–8 秒。

### 4.4 调试建议

- 在电脑浏览器登录教务系统并打开课表页，F12 控制台内粘贴脚本（去掉最外层的 `return ...`），执行后将结果赋给 `window.__scheduleDebugResult`，检查 `scheduleConfig`、`courses` 是否符合规范。
- 确认无误后，再改为「以 return 开头的完整注入代码」粘贴到管理后台「解析代码」中测试。

更细的 JS 注入约定（合并逻辑、常见报错）见同目录 **《课表解析-js注入-开发要求》**。

---

## 五、本地测试方法（App 内）

在脚本提交到管理后台或上线前，可在 **Expo 课表导入页** 内用「本地解析」功能在真机/模拟器上快速验证 HTML 解析与 JS 注入脚本，无需经过后端。

### 5.1 入口与前置条件

- **入口**：打开课表导入页面（从教务系统登录并进入课表页或至少加载过教务系统 URL），在页面**顶部导航栏右侧**找到 **问号图标（?）**，**长按**该图标即可打开「本地解析」弹窗。
- **前置**：
  - **HTML 解析**：无特殊要求，任意时刻均可测试（点击「尝试解析」时会先抓取当前 WebView 中的页面 HTML 再执行您的代码）。
  - **JS 注入**：必须先**在 WebView 内打开并加载好课表页面**（即已进入教务系统且课表内容已展示），否则会提示「请先加载课表页面」。

### 5.2 操作步骤

1. 长按问号图标，打开本地解析弹窗。
2. 选择解析方式：**HTML 解析** 或 **JS 注入**。
3. 在下方文本框粘贴或输入您的解析代码（与后台「解析代码」框中的内容一致）。
4. 点击 **「尝试解析」**：
   - **HTML 解析**：先从当前页面抓取 HTML（主框架 + 同源 iframe），再在 App 内嵌的隐藏 WebView 中执行 `function(html) { ... }`，执行约定与后端 `runHtmlCode` 一致。
   - **JS 注入**：在当前课表页面的 `window`（或同源 iframe 的 `contentWindow`）中执行您输入的代码，执行约定与四、JS 注入一致。
5. 查看结果：
   - 成功：显示解析到的课程数量，可点击 **「应用结果」** 进入课表预览并继续保存。
   - 失败：显示错误信息（如返回格式不符合、运行时异常等），可修改代码后再次点击「尝试解析」。

### 5.3 执行约定（与后端一致）

- **HTML 解析**：脚本以 `function(html) { ... }` 形式执行，必须 **return** 一个对象，且包含 `scheduleConfig` 与 `courses` 数组；格式同本文档「二、2.3」。
- **JS 注入**：整段代码必须以 **return** 开头（如 `return (function(){ ... })();`），返回值可为同步对象或 `Promise<{ scheduleConfig, courses [, timeNodes ] }>`；字段规范同「二、2.3」。

### 5.4 实现位置说明

本地解析逻辑位于前端 **`expo_app/app/schedule-import.tsx`**：

- 长按问号打开弹窗：导航栏问号按钮的 `onLongPress` 设置 `showLocalParseModal`。
- HTML 解析：抓取 HTML 后通过 `buildParserPageHtml(html, code)` 生成内嵌页面，在隐藏 WebView 中执行；结果通过 `localParseResult` / `localParseError` 的 postMessage 回传，由 `handleParserWebViewMessage` 处理。
- JS 注入：通过主 WebView 的 `injectJavaScript` 在课表页执行用户代码，结果通过 `injectionResult` / `injectionError` 回传；若存在 `localParsePendingRef.current?.type === 'js_inject'`，则走本地解析分支并展示结果，不请求后端。

---

以上为课表解析脚本的完整开发约定。若需示例代码，可参考项目内提供的示例规则文件；更多 JS 注入细节见 **《课表解析-js注入-开发要求》**。
