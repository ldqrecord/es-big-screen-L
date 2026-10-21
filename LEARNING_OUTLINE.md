# 大屏数据可视化与 Vue 3 全栈进阶学习大纲

> **目标受众**：具备扎实前端开发基础（熟悉 JS/TS/CSS/Web 基础），希望系统攻克 **Vue 3 组合式开发** 与 **企业级大屏数据可视化 (Big Screen Visualization)** 的开发者。  
> **配套项目代码**：[`es-big-screen`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen)  
> **学习模式**：循序渐进，先建立全局知识树与章节框架，后续随学习节奏逐步细化和填充每个章节的深度笔记。

---

## 📚 精简完整的课程章节大纲

```text
第 1 章：Vue 3 组合式 API 心智模型与核心语法（前端老手专攻）
  ├── 1.1 从 Options API 到 Composition API 的设计哲学与思维转变
  ├── 1.2 <script setup> 语法糖与 TypeScript 深度结合最佳实践
  ├── 1.3 核心响应式机制深究：ref vs reactive、toRefs、shallowRef 与响应式丢失陷阱
  ├── 1.4 副作用管理：watch、watchEffect 与 computed 的精准使用
  └── 1.5 现代化逻辑复用：组合式函数（Composables / 自定义 Hook）规范

第 2 章：大屏自适应架构方案剖析 (Scale 缩放体系)
  ├── 2.1 常见大屏分辨率基准与视觉设计规范（1920x1080、超宽带屏、拼接屏）
  ├── 2.2 大屏适配三大流派横评（rem + vw/vh、transform: scale()、百分比弹性网格）
  ├── 2.3 核心源码拆解：解析 src/utils/useResize.ts 实现等比与强制全屏算法
  └── 2.4 典型边缘坑点应对：scale 缩放后 Element-Plus/第三方弹窗/Tooltip 偏移错位解决方案

第 3 章：ECharts 5 工业级封装与图表体系实战
  ├── 3.1 ECharts 5 关键特性与按需加载引入策略
  ├── 3.2 核心组件封装解析：src/components/chart/Chart.vue 源码精读
  ├── 3.3 图表生命周期与内存安全：init、setOption、resize 监听与 dispose 释放
  ├── 3.4 大屏常用图表实战配置套路（双 Y 轴折柱混排、渐变色填充、炫酷饼图、雷达图）
  └── 3.5 交互与联动：图表高亮联动、点击穿透事件处理

第 4 章：大屏地理空间可视化 (GeoJSON 离线地图 + 高德地图)
  ├── 4.1 离线地理数据体系：GeoJSON 结构认知与 DataV 地理数据源获取
  ├── 4.2 ECharts Map 实现中国地图：省市钻取、视觉映射 (visualMap)、散点涟漪 (effectScatter)、迁徙飞线
  ├── 4.3 在线 WebGL 地图方案：@amap/amap-jsapi-loader 异步接入实战
  └── 4.4 大屏暗黑/科技风格高德自定义底图配置与 3D 楼块、轨迹线绘制

第 5 章：大屏特色动效与交互增强
  ├── 5.1 核心数据翻牌器：Odometer / Vue3-Odometer 滚动数字实现与千分位格式化
  ├── 5.2 告警与实时榜单：自定义无缝滚动 SeamlessScroll.vue 与 useSeamlessScroll 原理
  ├── 5.3 视觉震撼力引擎：利用 Anime.js 打造模块错落入场与呼吸光效
  └── 5.4 可视化工作台式交互：整合 es-drager 与 sortablejs 实现卡片自由拖拽与排版保存

第 6 章：大屏状态管理与实时数据流架构
  ├── 6.1 Pinia 最佳实战：src/store/screen.ts 管理主题样式、全局尺寸与控制台开关
  ├── 6.2 实时数据通讯：短轮询、长轮询 vs WebSocket 心跳与断线重连设计
  └── 6.3 前端 Mock 与数据适配器模式：如何优雅地把后端粗糙接口转换为 ECharts 标准结构

第 7 章：大屏 7×24h 生产级性能优化与运维排坑
  ├── 7.1 长时间驻留大屏防崩溃指南：Chrome DevTools Performance & Memory 排查内存泄漏
  ├── 7.2 渲染管线优化：Canvas vs SVG 模式选型、动画 requestAnimationFrame 调度
  └── 7.3 首屏加载极致提速：Vite 分包策略、Gzip 压缩与静态地图资源 CDN 化

第 8 章：项目全景源码实战串讲与独立二开实操
  ├── 8.1 梳理 src/views/screen/index.vue 大屏主舞台装配过程
  ├── 8.2 逐个攻克 Left1~Left2、Center、Right1~Right3 业务子模块
  └── 8.3 独立演练：从零搭建一个属于你自己的专属业务大屏
```
