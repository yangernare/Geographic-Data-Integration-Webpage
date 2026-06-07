地理空间数据导航与预处理平台 README

🌍 地理空间数据导航与预处理平台
聚合全球 50 + 地理空间数据平台，浏览器端完成遥感预处理与 AI 智能辅助
[功能特性](#-功能特性) • [快速开始](#-快速开始) • [使用说明](#-使用说明) • [技术架构](#-技术架构) • [数据源](#-数据源目录)


📖 项目介绍
项目 Logo
📍 Logo 位置: 请将项目 Logo 放置于 docs/assets/logo.png
地理空间数据导航与预处理平台是一个现代化的 Web 应用，致力于解决地理空间数据获取与预处理的痛点问题。平台聚合了全球 50 + 主流地理空间数据平台，中国平台优先展示，让用户能够快速定位并获取所需的数据资源。
基于前沿的 WebAssembly 技术，平台实现了纯浏览器端的遥感数据预处理，无需安装任何桌面软件即可完成格式转换、裁剪、重投影等专业操作。同时集成了本地大语言模型，提供自然语言交互能力，让数据查询和预处理变得更加智能便捷。
✨ 主要亮点
•	🔍 一站式数据导航 - 聚合 50 + 全球地理空间数据平台，中国平台优先展示
•	⚡ 浏览器端预处理 - 基于 GDAL WebAssembly，数据不上云，隐私安全
•	🤖 本地 AI 助手 - WebLLM 驱动，自然语言查询，离线可用
•	🚀 WebGPU 加速 - 充分利用现代 GPU 算力，处理效率提升数倍
•	🌐 完全离线可用 - 核心功能无需后端服务，纯前端运行
•	💻 跨平台支持 - 支持 Windows、macOS、Linux，无需环境配置

🎯 功能特性
1. 🗺️ 数据源导航
聚合 \\50+\\ 全球主流地理空间数据平台，采用中国平台优先的展示策略，帮助用户快速找到优质数据资源。
•	智能搜索 - 支持平台名称、数据类型、卫星传感器等多维度搜索
•	分类筛选 - 按数据类型（光学影像、SAR 雷达、DEM 高程、矢量数据等）筛选
•	中国优先 - 国内平台置顶展示，符合国内用户使用习惯
•	一键跳转 - 直接导航至对应平台，无需手动记忆网址
•	详细信息 - 每个平台包含数据类型、覆盖范围、访问条件等完整信息
2. 🛠️ 遥感预处理
基于GDAL 3.8.4 WebAssembly，在浏览器端完成专业的遥感数据预处理操作。
•	格式转换 - 支持 GeoTIFF、NetCDF、HDF、ENVI 等 20 + 格式互转
•	空间裁剪 - 按矢量边界或坐标范围精确裁剪影像
•	坐标重投影 - 支持 4000 + 坐标系统，包含 WGS84、CGCS2000、UTM 等
•	波段合成 - 多波段影像组合与分离
•	统计计算 - 像元统计、直方图分析、NDVI 等指数计算
•	植被指数 - 支持 NDVI、EVI、NDWI、SAVI 等多种植被指数计算
•	图像增强 - 直方图均衡化、对比度拉伸、锐化等增强处理
•	滤波处理 - 低通滤波（均值、高斯、中值）和高通滤波（拉普拉斯、Sobel、Prewitt、Canny）
•	数据隐私 - 所有处理在本地完成，数据无需上传服务器
3. 🧠 AI 智能助手
基于WebLLM技术，在浏览器端运行大语言模型，提供智能辅助功能。
•	自然语言查询 - "帮我找中国 30 米分辨率的 DEM 数据"
•	预处理指导 - "如何将 UTM 投影转换为 CGCS2000"
•	代码生成 - 自动生成 GDAL 命令或 Python 处理脚本
•	数据推荐 - 根据研究区域和主题推荐合适的数据源
•	完全离线 - 模型运行在本地，无需 API 调用，保护隐私

🚀 快速开始
环境要求
依赖	最低版本	推荐版本
Node.js	18.0.0	20.x LTS
npm	9.0.0	10.x
浏览器	Chrome 110+ / Edge 110+	Chrome 120+
安装步骤
1.	克隆项目仓库
bash
git clone https://github.com/your-org/geo-spatial-platform.git
cd geo-spatial-platform
2.	安装依赖
bash
npm install
3.	下载 GDAL WASM 模块（可选）
bash
npm run download-gdal
4.	启动开发服务器
bash
npm run dev
5.	访问应用
Plain Text
打开浏览器访问: http://localhost:5173
生产构建
bash
# 构建生产版本
npm run build

# 预览生产构建
npm run preview
Docker 部署
bash
# 构建镜像
docker build -t geo-spatial-platform .

# 运行容器
docker run -p 80:80 geo-spatial-platform

📚 使用说明
数据源导航模块
1.	浏览数据源
￮	首页默认展示所有数据源，中国平台置顶显示
￮	使用左侧分类面板按数据类型筛选
￮	使用顶部搜索框进行关键词搜索
2.	高级筛选
￮	点击「高级筛选」展开更多选项
￮	可按卫星系列、空间分辨率、时间范围等筛选
￮	支持多条件组合筛选
3.	查看详情
￮	点击数据源卡片查看完整信息
￮	包含平台介绍、数据类型、使用条件等
￮	点击「访问平台」直接跳转至官方网站
遥感预处理模块
1.	上传数据
￮	拖拽文件至上传区域或点击选择文件
￮	支持单文件或批量上传
￮	支持 GeoTIFF、HDF、NetCDF 等格式
2.	格式转换
￮	选择输出格式（GeoTIFF、PNG、JPEG 等）
￮	配置压缩选项和输出参数
￮	点击「开始转换」并等待处理完成
3.	空间裁剪
￮	上传 Shapefile 边界文件或手动绘制范围
￮	预览裁剪区域
￮	执行裁剪并下载结果
4.	重投影
￮	选择目标坐标系统（支持 EPSG 代码搜索）
￮	配置重采样方法
￮	执行重投影转换
AI 助手模块
1.	启动 AI 助手
￮	首次使用需下载 LLM 模型（约 4GB）
￮	模型加载完成后即可开始对话
￮	支持中文和英文交互
2.	示例对话
Plain Text
用户: 我需要云南省的30米DEM数据，有什么推荐？
AI: 推荐您使用地理空间数据云平台的SRTM 30米DEM数据...

用户: 如何将影像从WGS84转换为CGCS2000？
AI: 您可以使用预处理模块的重投影功能...
3.	代码生成
￮	描述您的处理需求
￮	AI 自动生成 GDAL 命令或 Python 脚本
￮	可直接复制使用

🏗️ 技术架构
核心技术栈
技术	版本	用途
GDAL WebAssembly	3.8.4	地理空间数据处理核心引擎
WebLLM	latest	浏览器端大语言模型运行时
geotiff.js	latest	GeoTIFF 文件解析与渲染
proj4js	latest	坐标系统转换
Tailwind CSS	3.4	现代化 UI 样式框架
WebGPU	latest	GPU 并行计算加速
系统架构图
Plain Text
┌─────────────────────────────────────────────────────────┐
│                    浏览器运行环境                         │
├─────────────┬─────────────────┬─────────────────────────┤
│             │                 │                         │
│  数据源导航  │   GDAL WASM     │      WebLLM             │
│  (纯前端)    │  (预处理核心)    │   (本地大模型)          │
│             │                 │                         │
├─────────────┴─────────────────┴─────────────────────────┤
│                                                         │
│              WebGPU 加速计算层                           │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│              Tailwind CSS + 现代UI组件                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
设计亮点
•	无服务端架构 - 核心功能纯前端实现，降低部署成本
•	渐进式 Web 应用 - 支持 PWA，可离线使用
•	模块化设计 - 各功能模块松耦合，易于扩展
•	响应式布局 - 适配桌面端和平板设备
•	国际化支持 - 内置中英文切换

🌐 浏览器支持
功能模块	Chrome	Edge	Firefox	Safari
数据源导航	✅	✅	✅	✅
GDAL 预处理	✅	✅	✅	✅
AI 助手	113+ ✅	113+ ✅	❌	❌
WebGPU 加速	113+ ✅	113+ ✅	❌	技术预览

注意: AI 助手功能需要浏览器支持 WebGPU，推荐使用 Chrome 113 + 或 Edge 113 + 版本以获得最佳体验。

📊 数据源目录
平台已收录 51 个 全球主流地理空间数据平台，按中国平台和国际平台分类展示。
🇨🇳 中国平台 (32 个)
序号	平台名称	运营机构	主要数据类型
1	地理空间数据云	中国科学院	Landsat、MODIS、高分系列、DEM
2	中国资源卫星应用中心	CRESDA	高分系列、资源系列、环境系列
3	天地图	自然资源部	影像底图、矢量底图、地形底图
4	风云卫星遥感数据服务网	国家卫星气象中心	风云二号 / 三号 / 四号气象卫星
5	中国气象数据网	中国气象局	地面气象、高空气象、卫星雷达
6	中国海洋卫星数据服务系统	国家卫星海洋应用中心	海洋一号 / 二号 / 三号系列卫星
7	国家海洋科学数据中心	国家海洋信息中心	海洋观测、水文、气象、遥感
8	海洋云（国家海洋大数据服务平台）	国家海洋信息中心	海洋生态、海域使用、海岛
9	自然资源部国土卫星遥感应用中心	LASAC	资源三号、高分系列卫星
10	国家遥感数据与应用服务平台	国家遥感中心	高分专项成果、多源遥感数据
11	国家地球系统科学数据中心	中科院地理所	土地利用、气象气候、水文
12	资源环境科学与数据中心	中科院地理所	行政区划、土地利用、NDVI
13	国家对地观测科学数据中心	中科院空天院	光学遥感、SAR 雷达、无人机
14	国家冰川冻土沙漠科学数据中心	中科院寒旱所	冰川编目、冻土、积雪
15	国家青藏高原科学数据中心	中科院青藏所	冰川变化、多年冻土、植被
16	国家地震科学数据中心	中国地震台网中心	地震波形、震相数据、地震目录
17	生态环境部卫星环境应用中心	生态环境部	大气、水、生态环境遥感监测
18	国家生态科学数据中心	中科院生态中心	植被、NDVI、碳密度、作物分布
19	国家林业和草原科学数据中心	国家林草局	森林资源、草原资源、湿地资源
20	地质云（中国地质调查局）	中国地质调查局	地质图、矿产、水文、工程地质
21	全国地理信息资源目录服务系统	国家基础地理信息中心	1:100 万 / 1:25 万基础地理信息
22	国家极地科学数据中心	中国极地研究中心	极地气象、冰川、海洋
23	国家空间科学数据中心	中科院国家空间科学中心	空间环境、太阳风、磁层
24	国家农业科学数据中心	中国农科院	作物种植分布、农业遥感监测
25	水利部数据服务平台	水利部	水资源、水文、江河湖泊
26	空基共性技术支撑服务网	中科院空天院	地表反射率、GNSS、叶面积指数
27	高分卫星 16 米数据共享平台	国家航天局	高分一号 / 六号 16 米全球数据
28	珞珈系列卫星数据服务	武汉大学	珞珈一号夜光、珞珈二号 Ka-SAR
29	中国水土保持监测网	水利部	水土流失监测、土壤侵蚀
30	中国遥感数据共享网	中科院空天院	Landsat 系列、国外商业卫星
31	GlobeLand30 全球地表覆盖数据	国家基础地理信息中心	30 米全球地表覆盖
32	中国科技资源共享网	科技部	20 家国家级科学数据中心入口

🌍 国际平台 (19 个)
序号	平台名称	国家 / 机构	主要数据类型
1	欧空局哥白尼数据空间	欧盟 / ESA	Sentinel 1/2/3/5P 全系列
2	美国地质调查局数据探索	美国 / USGS	Landsat 全系列、ASTER、SRTM
3	NASA 地球科学数据	美国 / NASA	MODIS、VIIRS、AIRS 等 80PB 数据
4	开放街道地图 (OSM)	开源社区	道路、建筑、水系、POI 矢量数据
5	自然地球 (Natural Earth)	开源	1:10m/50m/110m 全球地图数据
6	全球地球观测系统门户	GEO	全球多源地球观测数据
7	日本宇宙航空研究开发机构	日本 / JAXA	ALOS PALSAR、GCOM 系列
8	法国国家空间研究中心	法国 / CNES	SPOT、Pléiades、SWOT
9	美国国家海洋和大气管理局	美国 / NOAA	气象数据、海表温度、GOES
10	谷歌地球引擎	美国 / Google	80+PB 卫星影像，云端计算
11	微软行星计算机	美国 / Microsoft	云端地球科学数据与计算
12	亚马逊云开放数据	美国 / AWS	S3 托管 PB 级地球观测数据集
13	阿拉斯加卫星设施	美国 / NASA	Sentinel-1、ALOS PALSAR 等 SAR 数据
14	哥白尼气候数据商店	欧盟 / ECMWF	ERA5 再分析、气候数据集
15	开放地形 (OpenTopography)	美国	LiDAR 点云、高分辨率 DEM
16	社会经济数据与应用中心	美国 / NASA	人口分布、土地利用、环境指标
17	Radiant 机器学习中心	Radiant Earth	遥感机器学习标注数据集
18	Planet Labs	美国	每日更新 3-5 米高分辨率影像
19	Geofabrik OSM 数据提取	德国	按国家 / 地区的 OSM 数据提取

🤝 贡献指南
我们欢迎任何形式的贡献！无论是报告 Bug、提出功能建议还是提交代码，都非常感谢。
开始贡献
1.	Fork 本仓库
bash
# 点击GitHub页面右上角的Fork按钮
2.	克隆你的 Fork
bash
git clone https://github.com/your-username/geo-spatial-platform.git
cd geo-spatial-platform
3.	创建功能分支
bash
git checkout -b feature/your-feature-name
4.	提交更改
bash
git commit -m "Add: 描述你的更改内容"
5.	推送到分支
bash
git push origin feature/your-feature-name
6.	提交 Pull Request
贡献类型
•	🐛 Bug 修复 - 修复已知问题或发现新问题
•	✨ 新功能 - 添加新的数据源、预处理功能或 AI 能力
•	📚 文档 - 完善 README、使用文档或示例代码
•	🌐 国际化 - 翻译或改进多语言支持
•	🎨 UI/UX - 改进用户界面或交互体验
代码规范
•	使用 ESLint 进行代码检查
•	遵循 TypeScript 最佳实践
•	提交信息遵循 Conventional Commits 规范
•	新功能需包含相应的测试用例
添加新数据源
如果您希望添加新的数据源，请按以下格式提交信息：
yaml
名称: 平台全称
英文名称: 平台英文名称
运营机构: 运营单位
数据类型: [光学影像, SAR雷达, DEM高程, 矢量数据, ...]
访问条件: 免费/需注册/API
简介: 简短描述平台特色

📄 许可证
本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件。
Plain Text
MIT License

Copyright (c) 2024 地理空间数据导航与预处理平台

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

🙏 致谢
开源项目
本项目的诞生离不开以下优秀的开源项目：
•	GDAL - 地理空间数据处理的基石
•	WebLLM - 浏览器端 LLM 运行时
•	geotiff.js - GeoTIFF JavaScript 解析库
•	proj4js - 坐标转换 JavaScript 库
•	Tailwind CSS - 实用优先的 CSS 框架
数据平台
感谢所有开放地理空间数据的平台和机构，正是你们的开放共享精神推动了地理空间科学的发展。
特别感谢中国各国家级科学数据中心为科研工作者提供的优质数据资源。
贡献者
感谢所有为这个项目做出贡献的人！


如果这个项目对您有帮助，请给我们一个 ⭐ Star
Made with ❤️ by the Geo-Spatial Team

|（注：文档部分内容可能由 AI 生成）
