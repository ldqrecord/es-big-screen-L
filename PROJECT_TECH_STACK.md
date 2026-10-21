# es-big-screen 项目技术栈与架构深度解析

## 1. 项目定位与核心价值

- **项目定位**：轻量级、开箱即用的前端大屏数据可视化通用模板（基于 Vue 3 + Echarts + 高德地图 + Pinia）。
- **应用场景**：常规固定分辨率大屏监控中心（如智慧城市中控、电商大促监控看板、物流运输指挥中心）。
- **架构模式**：**纯前端单页应用（SPA）**，数据层通过静态 JSON 与 Mock 驱动，专注于大屏前端交互与视图展示。
- **核心差异化优势**：
  - **经典成熟的三栏大屏布局**：左侧核心指标监控、中间主地图与总览数据、右侧趋势与排行榜。
  - **优雅的大屏适配方案**：封装了极为简洁实用的 `useResize.ts`，通过 `transform: scale()` 自动等比缩放适配各种尺寸屏幕。
  - **双地图形态自由切换**：同时支持 Echarts 离线 GeoJSON 地图与高德地图（AMap JSAPI 2.0）无缝切换。
  - **动效与交互增强**：整合了数字翻牌器（Odometer）、入场动画（Anime.js）、图表卡片拖拽交换（Sortable.js + es-drager）以及表格/列表无缝向上平滑滚动（SeamlessScroll）。

---

## 2. 完整技术栈清单

| 分类 | 核心技术 / 依赖包 | 版本 | 用途说明 |
| :--- | :--- | :--- | :--- |
| **基础框架** | `Vue` | `^3.2.47` | 采用 Vue 3 `<script setup>` 组合式 API 开发 |
| **构建工具** | `Vite` | `^4.3.2` | 极速冷启动开发服务器与 Rollup 打包 |
| **语言规范** | `TypeScript` | `^5.0.2` | 类型检查与智能提示 |
| **状态管理** | `Pinia` | `^2.1.3` | 轻量化全局状态管理（存储屏幕尺寸、主题切换、图表配置状态） |
| **路由管理** | `vue-router` | `^4.2.1` | 前端路由跳转 |
| **UI 组件库** | `Element Plus` | `^2.3.7` | 辅助界面交互与图标组件 |
| **核心可视化** | `ECharts` | `^5.4.2` | 绘制折线图、柱状图、饼图、雷达图、中国散点/飞线地图 |
| **在线地图** | `@amap/amap-jsapi-loader` | `^1.0.1` | 官方高德地图 JSAPI 异步加载器（支持 3D 楼块、卫星图） |
| **卡片拖拽** | `es-drager` | `^0.3.5` | 基于 Vue 3 的网格/绝对位置拖拽组件（支持移动与缩放把手） |
| **列表排序** | `sortablejs` | `^1.15.0` | 经典的拖拽排序库，用于各模块卡片的拖动换位 |
| **数字翻牌动画** | `odometer` / `vue3-odometer` | `^0.4.8` / `^0.1.3` | 数字跳动翻牌特效组件（平滑数字递增动效） |
| **入场动画库** | `animejs` | `^3.2.1` | 强大的 JavaScript 动画引擎，驱动大屏各组件交错入场动画 |
| **样式预处理** | `Sass` | `^1.62.1` | SCSS 样式编写与变量组织 |
| **时间处理** | `dayjs` | `^1.11.8` | 大屏顶部时钟与日期格式化 |

---

## 3. 项目目录结构解析

```text
es-big-screen/
├── public/                       # 静态公共资源
├── src/
│   ├── assets/
│   │   ├── css/                  # 全局样式（base.scss、index.scss 重置与大屏样式）
│   │   ├── data/                 # 本地 Mock 数据（china.json 地图、hot、rank、seller、trend 等）
│   │   └── images/screen/        # 大屏装饰图（标题框、边框、切换按钮、深浅色图标）
│   ├── components/
│   │   ├── chart/                # 【核心组件】Echarts 通用封装组件（Chart.vue）
│   │   └── SeamlessScroll.vue    # 【核心组件】无缝无感知向上平滑滚动组件
│   ├── router/                   # 路由定义
│   ├── store/                    # Pinia 状态树（screen.ts 管理主题模式与全屏状态）
│   ├── utils/
│   │   ├── useResize.ts          # 【核心 Hook】窗口 resize 缩放自适应逻辑
│   │   ├── useSeamlessScroll.ts  # 【核心 Hook】无缝滚动底层动画驱动
│   │   └── useSortable.ts        # 【核心 Hook】拖拽排序绑定封装
│   ├── views/screen/             # 【核心大屏视图】
│   │   ├── components/
│   │   │   ├── Header.vue        # 顶部标题栏、时钟显示、全屏按钮、主题切换
│   │   │   ├── Title.vue         # 每个图表卡片的统一样式标题组件
│   │   │   ├── center/           # 中间区域（AMap.vue 高德地图、Map.vue Echarts 地图、Bottom.vue 核心计数指标）
│   │   │   ├── left/             # 左侧区域（Left1.vue 趋势分析、Left2.vue 商家分布）
│   │   │   └── right/            # 右侧区域（Right1.vue 销量排行、Right2.vue 热销占比、Right3.vue 库存统计）
│   │   └── index.vue             # 大屏主页面入口，组装所有子组件并挂载 useResize
│   ├── App.vue                   # 根组件
│   └── main.ts                   # 应用程序入口
├── index.html                    # 模板入口
├── package.json
└── vite.config.ts
```

---

## 4. 核心功能与架构设计亮点

### 1. 大屏自适应原理 (`useResize.ts`)
大屏项目最核心的技术难点在于“如何自适应各种不同分辨率的显示屏和拼接屏”。
本项目采用**标准设计稿固定尺寸 + CSS `transform: scale()` 动态等比缩放**方案：
- 默认基准尺寸设定为 `1920 × 1080`。
- 当浏览器窗口缩放时，计算 `scaleW = clientWidth / 1920` 和 `scaleH = clientHeight / 1080`。
- 比较屏幕宽高比，取较小比例进行等比居中缩放（避免失真拉伸）；若开启 `fullScreen` 则采用非等比强制铺满。
- 使用 `debounce` 防抖监听 `window.addEventListener('resize')`，大幅减轻重绘性能开销。

### 2. ECharts 统一封装规范 (`Chart.vue`)
- 将 ECharts 实例的初始化、销毁（`dispose`）、监听图表尺寸自适应（`chart.resize`）统一封装为一个通用组件。
- 业务页面只需通过 props 传入 `option` 配置项即可，解耦了 DOM 操作与业务逻辑。

### 3. 多卡片拖拽与动态换位
- 引入了作者自研的 `es-drager` 与 `sortablejs`，支持大屏各模块在运行态下根据运维需要进行拖拽和位置排序，增强了固定大屏的交互性。

---

## 5. 本地运行与开发指南

### 前提环境要求
- **Node.js**：`16.14.x` ~ `20.x`
- 包管理器推荐：`pnpm` 或 `yarn`

### 启动命令
```bash
# 1. 进入子目录
cd es-big-screen

# 2. 安装依赖（已提供 yarn.lock，推荐使用 yarn 或 pnpm）
yarn install
# 或者使用 npm
npm install

# 3. 启动本地开发服务
yarn dev

# 4. 构建生产产物
yarn build
```
本地服务启动后，在浏览器访问控制台提示的地址（默认通常为 `http://127.0.0.1:3006/` 或 `http://localhost:5173/`）。

---

## 6. 开发者学习重点推荐

1. **学习大屏自适应的黄金标准**：
   - 细读 `src/utils/useResize.ts`，理解其宽高比计算与 CSS 居中缩放实现，这是任何大屏项目最基础通用的底层代码。
2. **学习图表组件的规范封装**：
   - 查看 `src/components/chart/Chart.vue`，掌握如何优雅地管理 ECharts 实例的生命周期与动态自适应更新。
3. **学习大屏无缝滚动的实现**：
   - 查看 `src/utils/useSeamlessScroll.ts` 与 `SeamlessScroll.vue`，掌握利用 `requestAnimationFrame` 或 CSS 关键帧实现告警列表不卡顿滚动的技巧。
