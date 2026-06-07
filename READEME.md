# GeoNavigator - 地理空间数据导航与预处理网页应用

## 摘要

构建一个**单文件 HTML 应用**（`geonavigator.html`），面向 GIS/RS 专业人员和学生，提供三大核心功能：
1. **地理空间数据源聚合导航** — 收录 20+ 全球主流地理空间数据平台，支持搜索与分类筛选
2. **浏览器端遥感数据预处理** — 基于 gdal3.js（GDAL WebAssembly）实现格式转换、裁剪、重投影、波段提取、重采样、辐射定标、NDVI 计算等操作
3. **本地 AI 智能助手** — 基于 WebLLM（浏览器端 LLM）提供自然语言数据查询和预处理指导

整体采用**暗色主题**（黑色为主），所有计算均在浏览器端完成，无需后端服务器。

---

## 当前状态分析

从零开始构建，无现有代码库。需依赖以下 CDN 资源：
- **gdal3.js** — GDAL 3.8.4 的 WebAssembly 编译版本
- **@mlc-ai/web-llm** — 浏览器端 LLM 推理引擎（WebGPU 加速）
- **geotiff.js** — GeoTIFF 文件读写库
- **Tailwind CSS** — 响应式布局与样式框架
- 其他辅助库（proj4js、turf.js、marked.js、highlight.js、Font Awesome）

---

## 技术决策

| 决策项 | 选择 | 原因 |
|--------|------|------|
| gdal3.js 运行模式 | `useWorker: false`（主线程） | 单 HTML 文件无法内联 Worker 脚本文件，主线程模式是唯一可行方案 |
| NDVI 计算 | geotiff.js 逐像素计算 | gdal3.js 不包含 `gdal_calc`，需 JS 手动实现波段代数运算 |
| AI 模型 | Qwen2.5-3B-Instruct | 中文支持最好的小模型，~2.2GB VRAM，适合浏览器端运行 |
| 跨脚本通信 | `window.appState` 全局状态 | gdal3.js 用普通 `<script>` 加载，WebLLM 用 `<script type="module">`，需通过 window 共享状态 |
| GDAL 懒加载 | 用户首次切换到预处理 Tab 时初始化 | WASM 文件 ~25MB，避免首页加载过慢 |

---

## 页面布局设计

### 整体布局（三栏式）

```
+----------------------------------------------------------+
|                    顶部导航栏 (56px)                       |
|  [Logo] GeoNavigator    [数据源] [预处理] [关于]    [AI]   |
+----------+-------------------------------+---------------+
|          |                               |               |
| 侧边栏   |        主内容区                |  AI 助手面板   |
| (280px)  |                               |  (360px)      |
|          |                               |               |
| 搜索框   |  根据当前模块切换内容           |  聊天界面     |
| 分类筛选  |                               |               |
| 数据源列表|                               |               |
+----------+-------------------------------+---------------+
```

### 响应式断点

- **≥1280px**：三栏布局（侧边栏 + 主内容 + AI 面板）
- **768px~1279px**：侧边栏折叠为图标，AI 面板变为浮动按钮触发
- **<768px**：单栏布局，侧边栏和 AI 面板均为抽屉式覆盖层

### 暗色主题色板

```css
:root {
    --bg-primary: #0a0a0f;        /* 主背景 */
    --bg-secondary: #12121a;      /* 卡片/面板背景 */
    --bg-tertiary: #1a1a2e;       /* 悬停/激活背景 */
    --bg-elevated: #22223a;       /* 弹出层背景 */
    --border: #2a2a45;            /* 边框 */
    --text-primary: #e8e8f0;      /* 主文字 */
    --text-secondary: #8888a8;    /* 次要文字 */
    --accent-blue: #4f8ff7;       /* 主强调色 */
    --accent-green: #34d399;      /* 成功 */
    --accent-purple: #8b5cf6;     /* AI 相关 */
}
```

---

## 实施计划

### 阶段一：基础框架搭建

**目标**：完成 HTML 骨架、暗色主题、导航结构

1. 创建 HTML5 文档结构（meta 标签、viewport、lang="zh-CN"）
2. 引入所有 CDN 依赖（Tailwind CSS、Font Awesome、Google Fonts Noto Sans SC）
3. 定义 CSS 变量和暗色主题自定义样式
4. 构建顶部导航栏（Logo + 模块切换 Tab：数据源/预处理/关于 + AI 按钮）
5. 构建三栏布局骨架（侧边栏、主内容区、AI 面板）
6. 实现 Tab 切换逻辑（显示/隐藏对应模块）
7. 实现响应式布局（媒体查询 + Tailwind 断点）
8. 添加页面加载动画和过渡效果

**产出**：可运行的 HTML 骨架，暗色主题，Tab 切换正常

### 阶段二：数据源导航模块

**目标**：完成 20+ 数据源平台的展示、搜索、筛选

1. 定义 `DATA_SOURCES` 数组（20+ 平台完整数据结构，含 id、name、nameCN、url、description、categories、dataTypes、formats、regions、access、keywords、icon）
2. 定义 `CATEGORIES` 分类体系（dataType: 光学/SAR/DEM/矢量/气候/社会经济/LiDAR/ML；region: 全球/中国/亚洲/欧洲/美洲；access: 免费/需注册/API）
3. 实现数据源卡片组件渲染（平台名称、描述、数据类型标签、访问按钮）
4. 实现实时模糊搜索（对 name、nameCN、description、keywords、dataTypes 字段匹配）
5. 实现分类筛选（多选标签，同类别 OR、跨类别 AND）
6. 实现卡片详情展开/收起（详细描述、数据集列表、使用提示）
7. 实现搜索结果计数和空状态提示
8. 各平台外链（`target="_blank"`）

**收录平台清单**（20+）：
- ESA Copernicus Data Space、USGS EarthExplorer、NASA EarthData、地理空间数据云
- OpenStreetMap、Natural Earth、GEOSS、JAXA G-Portal、CNES GEODES、NOAA
- Google Earth Engine、Microsoft Planetary Computer、AWS Open Data、CRESDA
- ASF DAAC、Copernicus CDS、OpenTopography、SEDAC、天地图、Radiant MLHub、Planet Labs

**产出**：完整的数据源导航页面，支持搜索和筛选

### 阶段三：预处理模块 — GDAL 集成

**目标**：完成文件上传、元数据展示、基础 GDAL 操作

1. 加载 gdal3.js CDN（`https://cdn.jsdelivr.net/npm/gdal3.js@2.8.1/dist/package/gdal3.js`）
2. 实现 GDAL WASM 初始化（懒加载，含加载进度显示，`useWorker: false`）
3. 实现文件拖拽上传区（File API + Drag & Drop API）
4. 实现 `Gdal.open()` 读取文件 + `Gdal.getInfo()` 元数据解析与展示（文件名、格式、尺寸、波段数、分辨率、坐标系）
5. 实现格式转换功能（gdal_translate -of，支持 GTiff/COG/PNG/JPEG/ENVI 等）
6. 实现裁剪功能（gdal_translate -projwin 按范围裁剪）
7. 实现重投影功能（gdalwarp -t_srs）
8. 实现波段提取功能（gdal_translate -b）
9. 实现重采样功能（gdal_translate -r -tr，支持 nearest/bilinear/cubic/lanczos）
10. 实现辐射定标功能（gdal_translate -scale -expand）
11. 实现结果文件下载（Blob + createObjectURL）
12. 添加处理遮罩层（防止 UI 阻塞时用户误操作）

**产出**：完整的预处理工具页面，支持 7 种 GDAL 操作

### 阶段四：预处理模块 — NDVI 计算

**目标**：完成基于 geotiff.js 的 NDVI 计算

1. 加载 geotiff.js CDN
2. 实现双文件上传（红光波段 + 近红外波段）或单文件多波段选择
3. 实现波段数据读取（`readRasters()`）
4. 实现 NDVI 逐像素计算逻辑 `(NIR - Red) / (NIR + Red)`，含除零保护
5. 实现结果可视化（Canvas 渲染 NDVI 色带图）
6. 实现结果导出为 GeoTIFF（使用 geotiff.js 写出 + 原始元数据复制）

**产出**：NDVI 计算功能，含可视化预览和导出

### 阶段五：AI 助手模块

**目标**：完成 WebLLM 集成和智能对话

1. 创建 `<script type="module">` 区块
2. 动态 `import` WebLLM（`https://esm.run/@mlc-ai/web-llm`）
3. 实现 WebGPU 可用性检测（`navigator.gpu`），不支持时显示降级提示
4. 实现 MLCEngine 初始化（Qwen2.5-3B-Instruct，含模型下载进度条）
5. 实现聊天 UI（消息列表、输入框、发送按钮、Markdown 渲染）
6. 设计 System Prompt（地理空间数据专家角色，包含应用内置功能说明）
7. 实现流式输出（逐字显示回复）
8. 实现快捷问题按钮（预设 8 个常见问题）
9. 实现预处理上下文注入（AI 可感知当前文件信息，给出针对性建议）
10. 实现聊天历史管理（sessionStorage）
11. AI 面板的展开/收起/浮动按钮

**产出**：完整的 AI 助手面板，可回答地理空间相关问题

### 阶段六：集成优化与打磨

**目标**：模块间联动、性能优化、用户体验完善

1. 实现 AI 助手与预处理模块联动（AI 推荐预处理参数）
2. 实现 AI 助手与数据源模块联动（AI 推荐数据源）
3. 添加全局键盘快捷键（Ctrl+K 搜索、Ctrl+/ AI 面板）
4. 添加使用引导（首次访问提示）
5. 大文件处理优化（文件大小限制提示、内存警告）
6. 完善错误处理和友好提示（Toast 通知）
7. 添加"关于"页面（功能说明、技术栈、浏览器兼容性）
8. UI 最终打磨（动画过渡、间距微调、滚动优化）

**产出**：完整可交付的单文件 HTML 应用

---

## 关键技术挑战与应对

| 挑战 | 应对方案 |
|------|---------|
| 单文件中混合 `<script>` 和 `<script type="module">` | 通过 `window.appState` 全局对象共享状态 |
| GDAL WASM 首次加载 ~25MB | 懒加载 + 详细进度条 + 浏览器缓存提示 |
| WebGPU 兼容性（Firefox/Safari 不支持） | 检测 `navigator.gpu`，不支持时显示明确提示，其他功能不受影响 |
| 主线程 GDAL 操作阻塞 UI | 操作前显示全屏遮罩层 + `setTimeout(fn, 50)` 推迟一帧执行 |
| 大文件内存限制 | 文件大小上限提示（建议 <500MB），提供先缩小再处理的选项 |
| WebLLM 模型首次下载 ~2.2GB | 详细进度百分比 + 预估剩余时间 + 后续缓存秒开 |

---

## CDN 依赖清单

| 库 | 用途 | 加载方式 |
|---|---|---|
| Tailwind CSS | 响应式布局与样式 | `<script src="https://cdn.tailwindcss.com">` |
| Font Awesome 6.5 | 图标 | `<link>` CSS |
| Google Fonts Noto Sans SC | 中文字体 | `<link>` CSS |
| gdal3.js 2.8.1 | GDAL WebAssembly | `<script>` |
| geotiff.js | GeoTIFF 读写 | `<script>` |
| proj4js | 坐标系转换 | `<script>` |
| turf.js | 矢量空间分析 | `<script>` |
| marked.js | Markdown 渲染 | `<script>` |
| highlight.js | 代码高亮 | `<script>` |
| @mlc-ai/web-llm | 浏览器端 LLM | `<script type="module">` 动态 import |

---

## 验证步骤

1. **基础功能验证**：在 Chrome 113+ 中打开 HTML 文件，确认暗色主题正常渲染、Tab 切换正常
2. **数据源导航验证**：搜索"Sentinel"能找到 ESA Copernicus 和 ASF DAAC；筛选"SAR"只显示含 SAR 数据的平台；所有外链可正常打开
3. **预处理验证**：上传一个小型 GeoTIFF 文件，依次测试格式转换、裁剪、重投影、波段提取、重采样、辐射定标，确认输出文件可正常下载
4. **NDVI 验证**：上传含红光和近红外波段的影像，计算 NDVI 并验证结果色带图和导出文件
5. **AI 助手验证**：在支持 WebGPU 的浏览器中加载模型，测试预设快捷问题和自由对话，确认回复与地理空间相关
6. **兼容性验证**：在 Chrome、Edge、Firefox 中分别打开，确认数据源导航和预处理功能在所有浏览器正常，AI 面板在不支持 WebGPU 的浏览器中显示友好提示
7. **响应式验证**：缩放浏览器窗口至不同宽度，确认三栏/两栏/单栏布局切换正常
