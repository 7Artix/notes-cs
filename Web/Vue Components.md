将Vue Components放到 `App.vue` 中, 并从浏览器中预览.

一个 `.vue` 文件称为**SFC (Single File Component, 单文件组件)**. 将HTML, JS与逻辑, CSS封装到一起.

- `<template>` 是骨架HTML部分. 用于定义UI结构.
- `<script setup>` 是逻辑JS部分. `setup` 是语法糖(快捷方式). 加上 `setup` 后, 任何在script中定义的变量或函数, 可以直接在 `<template>` 中使用, 否则需要显式的 `return` .
- `<style scoped>` 是外观CSS部分. 用于定义样式. `scoped` 是参数, 保证样式只对当前组件生效. CSS是全局的, 加上 `scoped` 后, Vue会在编译时给标签生成一个唯一属性, 确保样式仅在单个组件内部有效.


# `<template>`

```vue
<template>
  <nav 
    class="navbar-container"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <div class="navbar-content">
      <!-- 左侧 Logo -->
      <div class="nav-logo" @click="navigateTo('/')">
        <div class="logo-placeholder">Apple</div>
      </div>

      <!-- 中间 菜单项 -->
      <ul class="nav-links">
        <li v-for="item in menuItems" :key="item.path" @click="navigateTo(item.path)">
          {{ item.name }}
          <!-- 悬停时显示的描述文字（扩展内容） -->
          <Transition name="fade">
            <span v-if="isHovered" class="nav-description">{{ item.desc }}</span>
          </Transition>
        </li>
      </ul>

      <!-- 右侧 头像 -->
      <div class="nav-avatar" @click="navigateTo('/me')">
        <img src="https://via.placeholder.com/40" alt="Avatar" />
      </div>
    </div>
  </nav>
</template>
```

简单的条件显示直接放置在 `<template>` 中. 使UI的结构与状态保持一致.

`<template>` 在页面加载时隐藏, 在稍后使用JavaScript显示. 在Vue中, 是组件的视图定义. Vue编译器会读取标签里的内容, 将其转换为JavaScript代码, 通过操作DOM(文档对象模型)显示.

`<template>` 里的表达式类似匿名闭包, 要求只能是**单个表达式**, 不能写复杂的条件循环.
若逻辑超过一行, 则应该在 `<script>` 中写成函数, 然后再在 `<template>` 中调用.

`key` Vue通过 `key` 识别元素, 操作DOM, 避免销毁重建, 而仅移动位置.

`<nav>` 导航栏标签.
	- `class` 用于分类, 主要用于和CSS匹配, 应用在CSS里对应的样式. 此外, 在复杂的JS中也可以通过 `class` 确定元素.
	- `@mouseenter` 在HTML中原本为 `v-on:click`. Vue为了简便缩写为 `@` . 后续执行了一个闭包操作, 进行了复制.
`<div>` 通用块/盒子.
`<ul>` Unordered List, 无序列表, 即列表前是原点, 不是数字.
`<li>` List Item, 列表中的某一项.
`<span>` 行内文本容器, 不会换行, 用于包裹一小段文字, 单独控制样式.
`<img>` 图片标签.

`<Transition>` 元素切换的样式, 可以通过CSS添加出入动态效果.


## `:` 和 `v-bind`

`v-` 开头的属性是Vue的缩写. 是Vue提供的指令, 通知Vue对标注的DOM元素作特殊处理.
- `v-for` 可以放置在任意标签上, 根据数组长度复制当前标签.
- `:` 是 `v-bind:` 的缩写. 是Vue**最关键的语法点**.
	- 不加冒号的 `key="item.path"` Vue会认为 `key` 的值是字符串字面量 `item.path`.
	- 加冒号的 `:key="item.path"` 代表引号内的是JavaScript变量, Vue会查找`item`对象下 `path` 变量的值.


# `<script>`

组件化开发, 与传统的线性程序在思维模式上有巨大差异.

传统C程序有固定的程序入口 `main()` .

在web项目中, 入口是 `src/main.js` .

随着Page开始渲染, 当浏览器渲染到一个组件时, 就是执行组件中编写的逻辑的触发点. 首先会执行 `<script setup>` 中的程序.

执行到 `watch` 进行**注册副作用** (Register Side Effect). 通知全局引擎登记监听器.

`watch` 绝对不是一个后台进程, JavaScript是单线程的, 也没有真正的并发. 其本质是**观察者模式 (Observer Pattern)**. 
当执行 `watch()` 时, Vue在 `isHovered` 对象内部维护了一个**回调函数列表 (Callback List)**. 
触发逻辑为: 修改 `isHovered.value = true` , 触发 `isHovered` 内部的 `Setter` 函数, 在 `Setter` 内部遍历回调函数列表, 执行对应的回调函数.

```vue

<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import gsap from 'gsap'

// 获取路由器实例, 钩子, 钩住全局的路由单例
const router = useRouter()
// 定义一个响应式Bool值, 均使用const, 因为变的是值对象, 不是isHovered
const isHovered = ref(true)

// 数组, 元素是对象Objects
const menuItems = [
  { name: 'Projects', path: '/projects', desc: 'My Works & Tools' },
  { name: 'Posts', path: '/posts', desc: 'Thoughts & Notes' },
  { name: 'CV', path: '/cv', desc: 'Experience & Skills' },
  { name: 'Playgrounds', path: '/playgrounds', desc: 'Lab & Fun' },
  { name: 'Search', path: '/search', desc: 'Find Content' },
]

// 定义函数(闭包)
const navigateTo = (path) => {
  console.log(`Navigating to: ${path}`)
  // 暂时先用 console.log，等配置好路由再启用 router.push(path)
}

// 观察者函数
watch(isHovered, (newVal) => {
  // newVal 是 isHovered 变化后的新值
  if (newVal) {
    // GSAP动画: 让class为'.navbar-container'的元素高度变化
    gsap.to('.navbar-container', { height: '120px', duration: 0.4, ease: 'power2.out' })
  } else {
    gsap.to('.navbar-container', { height: '60px', duration: 0.4, ease: 'power2.in' })
  }
})
</script>
```

`import` 导出, 可以具名导出 `{}` 导出独立的函数, 或者默认导出.

`ref` (Reference) API函数. 创建响应式引用. Vue为了能检测变量的变化, 将值包装成一个对象, 赋给变量. **响应式容器**.

`watch` 观察者, 监听某个 `ref` 值的变化, 一旦该值改变, 则执行一个回调函数 (Closure). 两个参数:
- 被观察对象 ( `isHovered` ).
- 变化是执行的回调函数 (闭包).

`useRouter` vue-router提供的Hook, 用于获取全局路由器的Handler.

Hook本质是**在特定时机'钩住'框架内部状态的方法**. 是框架提供的**功能入口**.
不同Context下的Hook的含义不相同 (例如在C语言中有另外的含义). 


## `=>` 箭头函数

`=>` **箭头函数 (Arrow Function)** 对应swift中的闭包语法 `{ (params) -> ReturnType in ... }` 结构为: `(参数) => { 函数体 }` 

当**没有**参数, 或有**两个及以上**参数时, 参数列表 `()` 必须保留.

```js
const sayHi = () => console.log('Hi');
const add = (a, b) => a + b;
```

当恰好仅有一个参数时, 括号可选.

```js
const double = n => n * 2;
const double = (n) => n * 2
```

箭头函数的返回值分为**显示返回**和**隐式返回**.

### 显式返回

```js
const add = (a, b) => {
  const result = a + b;
  return result; // 必须手动写 return, 否则返回 undefined
}
```

类比swift:

```swift
let add = { (a: Int, b: Int) -> Int in
    let result = a + b
    return result
}
```


### 隐式返回

隐式返回是箭头函数最常用的特性. 当函数仅有一个表达式, 可以省略 `{}` 和 `return` 关键字.

```js
const add = (a, b) => a + b;
```

类比swift:

```swift
let add = { $0 + $1 } // 自动返回表达式的结果
```


当需要返回对象, 例如 `{ name: 'John' }` 时, 错误写法为:

```js
const getProject = () => { name: 'John' };
```

此时会将 `{` 视为函数体的开始, 正确写法为:

```js
const getProject = () => ({ name: 'John' });
```


## JavaScript Object

JavaScript中的所有事物都是对象: 字符串, 数值, 数组, 函数...

例如数组在JavaScript中是:

```js
const arr = ['A', 'B'];
```

但在底层中, 实际是:

```js
{
  "0": "A",
  "1": "B",
  "length": 2,
  "push": function() { ... } // 继承自原型
}
```

操作对象时, 本质是在操作一组键值对.

- 对象(Object) = 属性名(Key) + 属性值(Value)
- 方法(Method) = 当属性值是函数对象时, 称之为方法

对象的本质是一个极其灵活的 `Hash Table` .
Key永远是字符串, 而Value可以是任何东西 (指针, 数值, 另一个哈希表...).
当Key是数字, 表现为数组, 当哈希表有可执行的代码块, 则表现为函数, 当有一些自定义属性, 则表现为结构体.

JavaScript是**动态弱类型语言**. JavaScript的变量没有类型, 但值有类型. JavaScript的变量只是一个指针, 可以指向内存中的任何东西.

```js
let x = 1;             // x 指向一个数字
x = "Hello";           // x 现在指向一个字符串
x = { name: "John" }; // x 现在指向一个对象（结构体）
x = () => { };         // x 现在指向一个函数
```

动态若类型语言, 在处理类似JSON数据(大字典), 这种随时可能增减字段的数据是, 比静态语言方便得多.

使用 TypeScript 后, 可以获得确定的变量类型.


# `<style>`

`<Template>` 决定有什么, `<style>` 决定怎么摆.

```vue
<style scoped>
.navbar-container { /* 外部容器, 包裹整个导航栏最外层方块 */
  position: fixed; /* 固定位置, 不随页面滚动 */
  top: 0; /* 定位坐标 */
  left: 0;
  width: 100%; /* 宽度撑满 */
  height: 60px; /* 初始高度 */
  background: rgba(255, 255, 255, 0.7); /* 背景颜色 */
  backdrop-filter: blur(10px); /* 高斯模糊, 模糊该元素后面的内容 */
  border-bottom: 1px solid rgba(255, 255, 255, 0.3); /* 底部边条, 增加光感 */
  z-index: 1000; /* Z轴深度, 窗口层级, 越大约在顶层 */
  display: flex; /* 开启Flexbox布局, 开启后子元素可以自由排版, 弹性布局 */
  justify-content: center; /* 在flex容器内的元素 (子元素) 水平居中 */
  transition: background 0.3s; /* 内置补间动画 */
  overflow: hidden; /* 裁剪 */
}

.navbar-content { /* 内部限制容器, 和最外层有padding */
  width: 90%;
  max-width: 1200px; /* 最大宽度限制, 防止显示器太大, 内容太散 */
  display: flex;
  justify-content: space-between; /* 和 flex 配合, 自动分配剩余空间, 中间元素居中 */
  align-items: center; /* 和 flex 配合 */
  height: 60px;
}

.nav-logo {
  cursor: pointer; /* 当鼠标悬停时, 指针变成手状 */
  font-weight: bold;
  font-size: 24px;
}

.nav-links { /* 列表父容器 */
  display: flex;
  list-style: none; /* 去掉 <ul> 默认自带的黑点列表符号 */
  gap: 40px; /* 子项之间间距, 需要开启flex, 仅在间隔产生, 最左右不产生 */
}

.nav-links li { /*选择器嵌套 选中 nav-links 内部所有 li 标签*/
  cursor: pointer;
  display: flex;
  flex-direction: column; /* 将 Flex 的排布方向设置为纵向 */
  align-items: center;
  font-weight: 500;
  color: #333;
}

.nav-description {
  font-size: 12px;
  color: #666;
  margin-top: 8px; /* 距离顶部边距 */
  white-space: nowrap; /* 禁止换行 */
}

.nav-avatar img {
  width: 40px; /* 固定尺寸 */
  height: 40px;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid #fff; /* 描边 */
}

/* 淡入淡出过渡 */
/* 当使用 <Transition name="fade"> 时, Vue会在显示/隐藏不同阶段, 给标签换为对应的类名 */
.fade-enter-active, .fade-leave-active { /* 动画执行期间 */
  transition: opacity 0.3s ease; /*不透明度在0.3s内以ease曲线变化*/
}
.fade-enter-from, .fade-leave-to { /*动画起始状态和结束状态*/
  opacity: 0;
}
</style>
```

`display: flex` 弹性布局. 能够扩展或收缩 `flex` 容器内的元素, 从而最大限度的填充可用空间.
任何容器均可指定为 `flex` 布局.

```css
.box{
  display: flex;
}
```

## `display: flex`

当设置为 `display: flex` 后, 子元素的 `float` , `clear` , `vertical-align` 属性失效.

可用于:
- 不同方向排列元素
- 重新排列元素显示顺序
- 更改元素对齐方式
- 动态将元素装入容器

容器存在水平主轴 (main axis), 垂直交叉轴 (cross axis).

可横可竖, 当切换为主轴为竖时, 对应交叉轴位横轴. `flex-direction: column`

`justify-content` 属性定义items在主轴上的对齐方式 (横向):

```css
.box {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

- `flex-start` 左对齐
- `flex-end ` 右对齐
- `center` 居中对齐
- `space-between` 两端对齐, 最左右两侧与边缘无空隙
- `space-around` 每个项目两侧的间隔相等, 最左右两侧与边缘有空隙


`align-items` 属性定义items在交叉轴上的对齐方式 (纵向):

```css
.box {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

- `flex-start` 上对齐
- `flex-end` 下对齐
- `center` 居中对齐
- `baseline` 项目第一行文字基线对齐
- `stretch` 拉伸填充


## 后代选择器

```css
.nav-links { /* 列表父容器 */
  display: flex;
  list-style: none; /* 去掉 <ul> 默认自带的黑点列表符号 */
  gap: 40px; /* 子项之间间距, 需要开启flex, 仅在间隔产生, 最左右不产生 */
}

.nav-links li { /*选择器嵌套 选中 nav-links 内部所有 li 标签*/
  cursor: pointer;
  display: flex;
  flex-direction: column; /* 将 Flex 的排布方向设置为纵向 */
  align-items: center;
  font-weight: 500;
  color: #333;
}
```

`.nav-links` 是类选择器, 匹配所有 `class="nav-links"` 的标签.
`li` 也是类选择器, 匹配所有 `<li>` 的标签.

后代选择器是将二者结合起来, 在所有类名为 `.nav-links` 的容器里，寻找所有的 `<li>` 对象.

也可以通过新标签来控制子项样式.


## `<Transition>` 状态机

将复杂动画拆解为状态转换.

当写 `<Transition name="fade">` 时, Vue会规定6个特殊类名.

进入阶段 (Enter), 对应"淡入"

1. `.fade-enter-from` : 起始点 (如透明度 0)
2. `.fade-enter-active` : 过渡过程 (如设置 0.3s 的过渡曲线)
3. `.fade-enter-to` : 结束点 (默认为 1)

离开阶段 (Leave), 对应"淡出"
1. `.fade-leave-from` : 起始点 (默认为1)
2. `.fade-leave-active` : 过渡过程 (如设置 0.3s 的过渡曲线)
3. `.fade-leave-to` : 结束点 (如透明度 0)

`<Transition name="fade">` 是Vue的动画管理方案, Vue和GSAP擅长的领域不同.

Vue适合管理是否存在, 即DOM的生命周期.

```vue
<Transition name="fade"> <!-- name的值是命名前缀 -->
    <span v-if="isHovered" class="nav-description">{{ item.desc }}</span>
</Transition>
```

这意味着, 当 `isHovered=false` 时, 这段文字在HTML中是彻底消失的. 内存释放.

GSAP擅长精确的物理运动, 即**改变属性数值**. 对已有元素的属性进行更新.

Vue/CSS不适合做这类动画, 因为CSS在处理此类动画时几乎无法实现平滑过渡, 以及打断等效果.

当Vue的动画无法满足时, 可以使用JavaScript钩子.

```vue
<Transition
  @before-enter="onBeforeEnter"
  @enter="onEnter" 
  @leave="onLeave"
>
  <div v-if="show">复杂动画</div>
</Transition>

<script setup>
import gsap from 'gsap'

// el 是正在被操作的 DOM 元素指针 (element) 任意参数名, el是惯例
// done 是函数指针, 是Vue传入的, 当完成后调用, 通知Vue结束.
const onEnter = (el, done) => {
  // 使用 GSAP 接管动画
  gsap.fromTo(el, 
    { x: -100, opacity: 0 }, 
    { x: 0, opacity: 1, duration: 1, onComplete: done }
  )
}
</script>
```

- `@before-enter`
- `@enter`
- `@after-enter`
- `@before-leave`
- `@leave`
- `@after-leave`

## `@` 和 `v-on`

在标准HTML中, 如要监听点击, 需要使用 `onclick="doSomething()"` .
在Vue中, 为了让变量变化和事件监听结合, 创造了**指令 (Directives)**.

`v-on` 是一个指令, 用来绑定事件监听器. 
语法为: `v-on:事件名="表达式或函数"` . 例如: 

```vue
v-on:click="count++"
v-on:mouseenter="handleHover"
```

`@` 是 `v-on` 的缩写, 缩写为 `@click` .

`@` 的事件大致分为两类, 
- HTML的原生标签: 例如 `@click` , `@keydown` , `@input` 等.
- Vue组件的自定义事件: 例如 `<Transition>` 的 `@enter` 等.


