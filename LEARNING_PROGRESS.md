<!-- LEARNING_SYNC_ANCHOR_START
project: es-big-screen
current_chapter: 1
current_subtopic: 1.1
status: in_progress
completed_chapters: []
last_updated: 2026-10-21 23:15:00
learner_profile: 前端开发工程师 (Vue3 & 大屏开发薄弱)
LEARNING_SYNC_ANCHOR_END -->

# 大屏数据可视化与 Vue 3 进阶学习进度与实战笔记

> **配套大纲**：参见 [`LEARNING_OUTLINE.md`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/LEARNING_OUTLINE.md)  
> **多设备同步规范**：每次完成章节学习后，本文件自动更新笔记并推进看板。在其他电脑学习时，执行 `git pull` 即可由 AI 解析顶部状态锚点无缝接续。

---

## 📌 总体学习进度跟踪看板

| 章节 | 核心主题 | 掌握度要求 | 学习状态 | 预计耗时 |
| :--- | :--- | :--- | :---: | :---: |
| **第 1 章** | [Vue 3 组合式 API 心智模型与核心语法](#第-1-章vue-3-组合式-api-心智模型与核心语法) | 掌握 `<script setup>`、响应式底层、自定义 Hook | 🟡 进行中 | 1.5 天 |
| **第 2 章** | [大屏自适应架构方案剖析 (Scale 缩放体系)](#第-2-章大屏自适应架构方案剖析-scale-缩放体系) | 吃透 `useResize.ts`、全屏 API、多分辨率适配 | 🟡 进行中 (核心原理已备) | 1 天 |
| **第 3 章** | [ECharts 5 工业级封装与图表体系实战](#第-3-章echarts-5-工业级封装与图表体系实战) | 掌握 `Chart.vue` 封装规范、防内存泄露、常用图表配置 | ⬜ 待开始 | 2 天 |
| **第 4 章** | [大屏地理空间可视化 (GeoJSON 离线地图 + 高德地图)](#第-4-章大屏地理空间可视化-geojson-离线地图--高德地图) | 离线地图下钻、飞线/散点层、在线高德 JSAPI 2.0 联动 | ⬜ 待开始 | 2 天 |
| **第 5 章** | [大屏特色动效与交互增强](#第-5-章大屏特色动效与交互增强) | 数字翻牌器、无缝平滑向上滚动、Anime.js 入场动效、拖拽换位 | ⬜ 待开始 | 1.5 天 |
| **第 6 章** | [大屏状态管理与实时数据流架构](#第-6-章大屏状态管理与实时数据流架构) | Pinia 集中管理、长轮询/WebSocket 机制、Mock 与数据转换 | ⬜ 待开始 | 1 天 |
| **第 7 章** | [大屏 7×24h 生产级性能优化与运维排坑](#第-7-章大屏-724h-生产级性能优化与运维排坑) | 内存泄露排查、重绘/重排节流、DOM/SVG vs Canvas 抉择 | ⬜ 待开始 | 1 天 |
| **第 8 章** | [项目全景源码实战串讲与独立二开实操](#第-8-章项目全景源码实战串讲与独立二开实操) | 独立开发/重构一个定制主题大屏项目 | ⬜ 待开始 | 2 天 |

---

## 📝 逐章实战笔记与避坑速查

---

### 第 1 章：Vue 3 组合式 API 心智模型与核心语法

#### 1.1 从 Options API 到 Composition API
- **传统 Options API (Vue 2) 痛点**：单个功能的数据、方法、计算属性被强制切碎在 `data`、`methods`、`computed`、`mounted` 中，业务变大后上下跳转排查困难；`mixins` 存在变量名冲突与隐式来源问题。
- **Composition API 优势**：**基于逻辑关注点组织代码**，同一功能的响应式数据与方法聚合在一起，易于抽离为可复用的自定义 Hook（Composables），且具备一流的 TypeScript 类型推导。

#### 1.2 `<script setup>` 核心范式
- 顶层声明的变量、函数、`import` 自动暴露给模板，**无需 `return`**。
- 引入组件自动注册，无需 `components: { ... }` 声明。

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const count = ref<number>(0)
const doubleCount = computed(() => count.value * 2)

function increment() {
  count.value++
}

onMounted(() => {
  console.log('组件已挂载，初始计数值：', count.value)
})
</script>

<template>
  <button @click="increment">当前计数: {{ count }}，双倍: {{ doubleCount }}</button>
</template>
```

#### 1.3 `ref` vs `reactive` 与避坑清单
- **`ref`**：
  - 支持所有类型（基本类型与对象）；
  - JS/TS 中必须通过 `.value` 访问，template 中自动解包；
  - **最佳实践**：**业务开发优先全量使用 `ref`**，心智负担小，不易丢失响应性。
- **`reactive`**（仅限对象类型）：
  - ⚠️ **避坑 1（解构丢失响应式）**：
    ```ts
    const state = reactive({ x: 1, y: 2 })
    const { x } = state // ❌ x 变成普通数值，失去响应式！
    // ✅ 正确做法：使用 toRefs
    import { toRefs } from 'vue'
    const { x: safeX } = toRefs(state)
    ```
  - ⚠️ **避坑 2（重新赋值破坏代理）**：
    ```ts
    let userInfo = reactive({ name: 'Tom' })
    userInfo = { name: 'Jerry' } // ❌ 响应式代理彻底断开！
    ```

#### 1.4 自定义 Hook（Composables）思想
- 命名规范：以 `useXxx` 开头，如本项目中的 [`src/utils/useResize.ts`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/src/utils/useResize.ts)。
- 抽离共性状态与副作用逻辑，返回响应式数据或控制方法。

---

### 第 2 章：大屏自适应架构方案剖析 (Scale 缩放体系)

#### 2.1 常见大屏适配方案对比

| 适配方案 | 实现原理 | 优点 | 缺点 | 适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **`transform: scale()`** (本项目使用) | 基于基准分辨率（如 1920×1080），动态计算缩放比例并变换父容器 | 极简、全模块等比缩放、无需单位转换 | 弹窗/Tooltip 脱节、文字偶尔轻微发虚 | **常规/企业级大屏首选** |
| **`rem + vw/vh`** | `postcss-pxtorem` + 动态改根字体 | 真实 DOM 流，不破坏弹窗定位 | ECharts 内 `fontSize`/`padding` 需额外适配 | 混合型大屏、移动端响应式 |
| **百分比弹性网格** | Flexbox / Grid 流式排版 | 适应超大宽比拉伸 | 复杂图形视觉比例容易失真 | 纯数据后台报表 |

#### 2.2 本项目 `useResize.ts` 源码核心公式
核心实现位于 [`src/utils/useResize.ts`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/src/utils/useResize.ts)：

```typescript
export const width = 1920
export const height = 1080

const resize = () => {
  if (!screenRef.value) return
  const clientWidth = document.body.clientWidth
  const clientHeight = document.body.clientHeight

  const scaleW = clientWidth / width
  const scaleH = clientHeight / height

  if (fullScreen) {
    // 强制铺满全屏（无黑边，可能产生微变形）
    screenRef.value.style.transform = `scale(${scaleW}, ${scaleH})`
  } else {
    // 等比居中缩放（画面比例不变，留安全黑边）
    const scale = Math.min(scaleW, scaleH)
    screenRef.value.style.transform = `scale(${scale}) translate(-50%, -50%)`
    screenRef.value.style.top = '50%'
    screenRef.value.style.left = '50%'
  }
}
```

#### 2.3 核心避坑点：Scale 缩放体系下的浮层与弹窗偏移
- **问题根源**：外部容器做了 `transform: scale()`，而第三方弹窗（如 Element Plus 的 `el-select`、`el-tooltip`）默认通过 `teleport` 挂载在 `document.body` 上，无法感知缩放矩阵，导致悬浮错位。
- **解决方案**：
  1. 将浮层挂载节点设置在 Scale 缩放容器内（如 Element Plus 配置 `:append-to="screenRef"` 或 `teleported: false`）；
  2. ECharts 的 Tooltip 统一添加 `confine: true`，限制在图表容器内部渲染。

---

### 第 3 章：ECharts 5 工业级封装与图表体系实战
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] 掌握 `src/components/chart/Chart.vue` 组件传参 `options` 与防抖 `chart.resize()` 实现；
> - [ ] 掌握为什么组件卸载时必须调用 `chartInstance.dispose()`；
> - [ ] 折线图阴影渐变（`new echarts.graphic.LinearGradient`）调优；
> - [ ] 柱状图象形柱图与胶囊条设计；
> - [ ] 多系列图表的数据对齐机制。

---

### 第 4 章：大屏地理空间可视化 (GeoJSON 离线地图 + 高德地图)
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] 解析 `src/assets/data/china.json` 地图坐标注册机制 `echarts.registerMap('china', chinaJson)`；
> - [ ] 掌握散点涟漪动效 `type: 'effectScatter'` 与飞线动效 `type: 'lines'`；
> - [ ] 掌握 `@amap/amap-jsapi-loader` 异步加载机制与安全密钥配置；
> - [ ] 地图与左右侧卡片数据的联动逻辑。

---

### 第 5 章：大屏特色动效与交互增强
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] `vue3-odometer` 数字翻牌器在 Vue3 中的双向绑定技巧；
> - [ ] `SeamlessScroll.vue` 利用 `requestAnimationFrame` 实现平滑滚动并支持鼠标悬浮暂停；
> - [ ] `animejs` 入场交错延迟（stagger）动效配置；
> - [ ] `es-drager` 与 `sortablejs` 拖拽换位状态保存。

---

### 第 6 章：大屏状态管理与实时数据流架构
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] `src/store/screen.ts` 中的 Pinia Store 组织形式与 actions；
> - [ ] 大屏定时任务的统一注册、统一销毁与页面可见性监听（Page Visibility API）；
> - [ ] WebSocket 连接异常重试策略。

---

### 第 7 章：大屏 7×24h 生产级性能优化与运维排坑
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] Chrome Performance 帧率分析；
> - [ ] 定时器残留与 ECharts 实例泄漏实地排查；
> - [ ] 大屏字体（如数码晶体管字体）引入优化与体积瘦身。

---

### 第 8 章：项目全景源码实战串讲与独立二开实操
> ⏳ **[待跟进学习后填充具体笔记]**  
> *学习清单预留*：
> - [ ] 完整梳理主入口 `src/views/screen/index.vue` 的 DOM 层级与样式布局；
> - [ ] 替换一套属于自己业务线的数据并重构一个专属可视化大屏。
