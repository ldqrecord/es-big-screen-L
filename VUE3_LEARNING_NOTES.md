# Vue 3 核心心智模型与语法对比速查手册

> **定位**：大屏项目专属 Vue 3 专项进阶知识库。  
> **编写宗旨**：拒绝晦涩八股文，聚焦“生活比喻 + Vue 2 (Options / Class TS) 对应映射 + 项目真实代码切片 + 踩坑防雷”。

---

## 📌 目录
- [1. 核心心智模型转换：从 Options/Class 到 Composition](#1-核心心智模型转换从-optionsclass-到-composition)
- [2. 响应式核心声明：`ref` 与 `.value` 智能盒机制](#2-响应式核心声明ref-与-value-智能盒机制)
- [3. 计算属性：`computed` 的多种写法与 Vue 2 Class TS 映射](#3-计算属性computed-的多种写法与-vue-2-class-ts-映射)
- [4. 全局状态管理：Pinia vs Vuex 架构对比](#4-全局状态管理pinia-vs-vuex-架构对比)
- [5. 生命周期钩子映射表（大屏防内存泄漏重点）](#5-生命周期钩子映射表大屏防内存泄漏重点)
- [6. 本项目经典组件源码精析（持续追加中...）](#6-本项目经典组件源码精析持续追加中)
- [7. 组件通信新姿势：`defineProps` 与 `defineEmits`](#7-组件通信新姿势defineprops-与-defineemits)
- [8. `ref` vs `reactive` 与“解构丢失响应式”大坑](#8-ref-vs-reactive-与解构丢失响应式大坑)
- [9. 侦听器：`watch` 与 `watchEffect`（大屏图表联动核心）](#9-侦听器watch-与-watcheffect大屏图表联动核心)

---

## 1. 核心心智模型转换：从 Options/Class 到 Composition

### 1.1 三种开发范式横向对比

| 维度 | Vue 2 传统 Options API | Vue 2 + TypeScript (Class Component) | Vue 3 `<script setup>` (本项目采用) |
| :--- | :--- | :--- | :--- |
| **代码组织** | 按选项切碎（`data` / `methods` / `computed`） | 类属性与装饰器（`@Component`, `@Prop`） | **按业务逻辑关注点聚合**，自由组合 |
| **TS 类型推导** | 极度困难，常需 `this as any` | 依靠 TypeScript 类的静态类型 | **天然原生支持一流类型推导** |
| **逻辑复用机制** | `mixins`（命名冲突、来源黑盒） | 装饰器或多重继承（过于厚重） | **自定义 Hook（Composables）** |
| **样板代码量** | 偏多（必须包裹大对象） | 很多（装饰器语法与类样板） | **最少**（引入即用，无 `return`，无组件注册） |

---

## 2. 响应式核心声明：`ref` 与 `.value` 智能盒机制

### 2.1 什么是响应式？为什么要用 `ref`？
在原生 JavaScript 中，普通变量是一个纯粹的数值或字符串：
```js
let currentTime = ''
currentTime = '2026-10-26 22:30:00' // 界面 DOM 是“瞎子”，根本感知不到变量变了！
```
而在 Vue 3 中：
```ts
const currentTime = ref('')
```
- **生活比喻**：`ref('')` 就是把空字符串装进了一个**“带铃铛的智能快递盒”**（`RefImpl` 实例）。
- **读写规则**：
  - **在 JS 脚本中**：必须打开盖子读写内部值，即 `currentTime.value = '2026-10-26 22:30:00'`。一旦给 `.value` 赋值，盒子上的**铃铛就会响**，Vue 就会精确通知使用过它的 DOM 节点重新渲染！
  - **在 HTML 模板中**：`<span class="datetime">{{ currentTime }}</span>`，Vue 编译器在生成渲染函数时会自动进行“解包（Unwrap）”，因此**模板中严禁写 `.value`**。

### 2.2 三种写法的演进对比

#### 场景：声明一个响应式字符串 `currentTime`
```ts
// 1. Vue 2 Options API
data() {
  return {
    currentTime: ''
  }
}
// 使用：this.currentTime = '...'

// 2. Vue 2 Class Component (TS)
@Component
export default class Header extends Vue {
  currentTime: string = ''
}
// 使用：this.currentTime = '...'

// 3. Vue 3 Composition API (本项目)
const currentTime = ref<string>('')
// 使用：currentTime.value = '...'
```

---

## 3. 计算属性：`computed` 的多种写法与 Vue 2 Class TS 映射

### 3.1 常见写法 1：只读计算属性（箭头函数最简写法，本项目采用）

本项目 [`src/views/screen/components/Header.vue`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/src/views/screen/components/Header.vue) 源码：
```ts
const icon = computed(() => store.theme === 'dark' ? darkIcon : lightIcon)
```
- 只要 `store.theme` 改变，`icon` 就会自动重新计算并缓存；如果依赖项未变，多次读取直接命中缓存。

### 3.2 常见写法 2：带 `getter` 和 `setter` 的可写计算属性

当计算属性不仅需要被展示，还需要通过 `v-model` 或手动赋值去倒推修改源数据时，可以传入一个包含 `get` 和 `set` 的对象：
```ts
import { ref, computed } from 'vue'

const firstName = ref('张')
const lastName = ref('三')

// 可读可写的计算属性
const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newVal: string) {
    const names = newVal.split(' ')
    firstName.value = names[0] || ''
    lastName.value = names[1] || ''
  }
})

// 修改计算属性会触发 setter：
fullName.value = '李 四' 
// 此时 firstName.value 变成 '李'，lastName.value 变成 '四'
```

### 3.3 计算属性新旧语法全景映射（学员重点关切）

#### 场景：根据 `theme` 动态计算 `icon`
```ts
// -------------------------------------------------------------
// 【形态 1：Vue 2 Options API】
export default {
  computed: {
    icon() {
      return this.theme === 'dark' ? darkIcon : lightIcon
    }
  }
}

// -------------------------------------------------------------
// 【形态 2：Vue 2 + TS Class Component（你记忆中的写法！）】
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class Header extends Vue {
  theme: string = 'dark'

  // 在 TS 类写法中，计算属性正是用原生的 ES6 getter 实现的！
  get icon(): string {
    return this.theme === 'dark' ? darkIcon : lightIcon
  }

  // 如果需要可写计算属性，配对写 setter：
  set icon(val: string) {
    // ...
  }
}

// -------------------------------------------------------------
// 【形态 3：Vue 3 Composition API（本项目写法）】
import { computed } from 'vue'

// 只读形式
const icon = computed(() => store.theme === 'dark' ? darkIcon : lightIcon)

// 完整 get/set 形式
const icon = computed({
  get: () => store.theme === 'dark' ? darkIcon : lightIcon,
  set: (val) => { /* 处理赋值 */ }
})
```

---

## 4. 全局状态管理：Pinia vs Vuex 架构对比

### 4.1 `const store = useScreenStore()` 到底是什么？
**完全正确！Pinia 就是 Vue 3 官方专门用来取代 Vuex 的新一代状态管理库**（业内俗称 Vuex 5）。

在项目入口中可以看到：
```ts
import { useScreenStore } from '@/store'
const store = useScreenStore()
```

### 4.2 Pinia 与 Vuex 差异核心对比

| 核心特性 | Vue 2 + Vuex (3.x/4.x) | Vue 3 + Pinia (本项目采用) | 带来的体验飞跃 |
| :--- | :--- | :--- | :--- |
| **Mutations** | **必须有**（必须通过 `commit('mutationName')` 修改数据） | **彻底废弃**！直接修改即可 | 不再需要为改一个变量写三层跳转 |
| **修改状态方式** | `this.$store.commit('setTheme', 'light')` | `store.theme = 'light'` 或 `store.$patch({ theme: 'light' })` | 像操作普通对象一样直接友好 |
| **异步操作** | 必须写 `actions` 配合 `dispatch` | `actions` 中直接支持 `async/await`，不区分同步异步 | 心智模型统一 |
| **TypeScript 支持** | 极度羸弱，字符串类型无补全，常配 helper 库 | **开箱即用完美推导**，打点直接补全字段 | 代码重构时不怕拼写错误 |
| **使用方式** | 全局单例注入 `this.$store` | 模块化导出 Hook，如 `useScreenStore()` | 按需引入，体积更轻 |

### 4.3 项目源码中 Pinia 的使用方式实切片
查看 [`src/views/screen/components/Header.vue:L30-L34`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/src/views/screen/components/Header.vue#L30-L34)：
```ts
function handleChangeTheme() {
  // 使用 Pinia 提供的 $patch 批量修改状态
  store.$patch({
    theme: store.theme === 'dark' ? 'light' : 'dark'
  })
}
```

---

## 5. 生命周期钩子映射表（大屏防内存泄漏重点）

在 7×24 小时不间断运行的大屏项目中，定时器、图表实例、窗口 Resize 监听器的销毁至关重要。

| Vue 2 生命周期 | Vue 3 组合式 API 生命周期 | 常见应用场景与避坑要点 |
| :--- | :--- | :--- |
| `beforeCreate` / `created` | **不需要！** 直接写在 `<script setup>` 顶层即可 | 请求初始数据、读取路由参数 |
| `beforeMount` | `onBeforeMount` | 挂载前准备 |
| `mounted` | `onMounted` | 获取 DOM 节点（如 ECharts 初始化 `echarts.init(dom)`、绑定 Resize 监听） |
| `beforeDestroy` | `onBeforeUnmount` (本项目重点) | **人走关灯**：`clearTimeout`、`chart.dispose()`、`removeEventListener` |
| `destroyed` | `onUnmounted` | 彻底销毁后的收尾工作 |

---

## 6. 本项目经典组件源码精析（持续追加中...）

### 6.1 `Header.vue` 20 行极简时钟
```vue
<script setup lang='ts'>
import { computed, onBeforeUnmount, ref } from 'vue'
import dayjs from 'dayjs'
import { useScreenStore } from '@/store'

// 1. 获取 Pinia 全局状态
const store = useScreenStore()

// 2. 计算属性：根据主题动态返回对应图标路径
const icon = computed(() => store.theme === 'dark' ? darkIcon : lightIcon)

// 3. 响应式时间字符串
const currentTime = ref('')
const timeId = ref()

// 4. 定时时钟：递归 setTimeout
function startTime() {
  timeId.value = setTimeout(() => {
    currentTime.value = dayjs().format('YYYY-MM-DD HH:mm:ss')
    startTime()
  }, 1000)
}

// 5. 离开页面前清理定时器，杜绝后台死循环
onBeforeUnmount(() => {
  clearTimeout(timeId.value)
})

startTime()
</script>
```

---

## 7. 组件通信新姿势：`defineProps` 与 `defineEmits`

### 7.1 核心心智：编译器宏（Compiler Macro）
在 Vue 3 `<script setup>` 中，`defineProps` 和 `defineEmits` 是**编译器宏命令**：
- **不需要手动 `import`**！Vue 编译器在编译代码时会自动识别并处理；
- 接收父组件传入的数据就像声明普通变量一样直接。

### 7.2 三代写法全景映射

#### 场景：子组件声明一个 `title` 属性
```ts
// -------------------------------------------------------------
// 【形态 1：Vue 2 Options API】
export default {
  props: {
    title: {
      type: String,
      default: ''
    }
  }
}

// -------------------------------------------------------------
// 【形态 2：Vue 2 Class Component (TS 装饰器)】
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class Title extends Vue {
  @Prop({ type: String, default: '' })
  readonly title!: string
}

// -------------------------------------------------------------
// 【形态 3：Vue 3 <script setup> (本项目写法)】
// 方式 A：运行时声明（查看 src/views/screen/components/Title.vue）
const props = defineProps({
  title: String
})

// 方式 B：纯 TS 类型声明（更推荐，极其清爽！）
const props = defineProps<{
  title?: string
}>()
```

---

## 8. `ref` vs `reactive` 与“解构丢失响应式”大坑

### 8.1 为什么很多 Vue 2 开发者转 Vue 3 会踩坑？
很多习惯了 Vue 2 `data() { return { x: 1, y: 2 } }` 的开发者，初学 Vue 3 时特别喜欢用 `reactive`：
```ts
const user = reactive({ name: 'Tom', age: 18 })
```
然而，当你想简写代码时：
```ts
// ❌ 灾难现场：直接解构！
const { name, age } = user 

// 此时 name 变成了普通字符串 'Tom'，age 变成了普通数字 18！
// 你再修改 user.age++，界面上的 age 纹丝不动，响应式彻底断开！
```

### 8.2 拯救方案：`toRefs`
如果一定要解构 `reactive` 对象，必须用 `toRefs` 包裹：
```ts
import { reactive, toRefs } from 'vue'

const user = reactive({ name: 'Tom', age: 18 })
// ✅ 正确做法：把对象的每一个属性都转换成独立的 ref
const { name, age } = toRefs(user)

age.value++ // 界面正常响应！
```

> [!TIP]
> **官方与企业级大屏最佳实践**：
> - 基础类型（字符串、数字、布尔值）和 DOM 引用：**一律用 `ref()`**；
> - 只要遇到解构场景，要么直接全用 `ref`，要么必须使用 `toRefs`，绝不裸解构。

---

## 9. 侦听器：`watch` 与 `watchEffect`（大屏图表联动核心）

### 9.1 大屏图表为什么离不开 `watch`？
查看 [`src/components/chart/Chart.vue:L35-L38`](file:///g:/学习/工作/AI项目/大屏项目/es-big-screen/src/components/chart/Chart.vue#L35-L38)：
```ts
watch(() => props.option, () => {
  setOption(props.option)
})
```
- 父组件的图表数据经常是异步请求或者定时轮询更新的（比如每隔 10 秒拿一批新数据）；
- 子组件 `Chart.vue` 通过 `watch` 死死盯着 `props.option`，一旦父组件更新了数据，子组件立刻调用 `chart.setOption()` 重绘图表。

### 9.2 `watch` vs `watchEffect` 的直观差别

| 特性 | `watch` (精准监控探针) | `watchEffect` (自动依赖收集) |
| :--- | :--- | :--- |
| **工作原理** | 必须显式告诉它“盯住谁”：`watch(source, callback)` | 不用告诉它盯谁，执行体里用到谁就自动盯谁 |
| **首次执行** | 默认惰性执行（初始不执行，数据变了才执行） | 立即自动执行一次（为了收集依赖） |
| **新旧值** | 可以同时获取 `(newVal, oldVal)` | 拿不到旧值，只关心当前最新状态 |
| **适用场景** | 监听 `props` 变化重绘图表、监听分页变化发请求 | 根据多个变量自动合成某个副作用并立即生效 |

#### 对比示例：
```ts
// 【watch 写法】：明确指定盯住 props.option
watch(() => props.option, (newOpt, oldOpt) => {
  console.log('配置更新了', newOpt)
  chart.setOption(newOpt)
})

// 【watchEffect 写法】：直接写逻辑，里面用了谁就自动监听谁
watchEffect(() => {
  console.log('当前主题是：', store.theme) // 只要 store.theme 变了就会触发
})
```

