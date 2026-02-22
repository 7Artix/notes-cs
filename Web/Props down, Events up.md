属性下行, 事件上行. 是现代前端框架中组件通信的核心原则.

规定了父组件和子组件之间信息传递的交通规则.

# Props Down

父组件通过 `props` 向子组件传递数据.

- 单向性: 数据流是单向的, 从父组件流向子组件.
- 只读性: 子组件不应该直接修改收到的 `props` .
- 目的: 保证数据的唯一性, 便于调试.

例如, 父组件有一本书的名字, 传递给子组件:

```vue
<!-- Parent.vue (父组件) -->
<template>
  <ChildComponent book-name="Vue 进阶指南" />
</template>
```

```js
// ChildComponent.vue (子组件)
const props = defineProps(['bookName']) 
// 子组件通过 props 接收到 "Vue 进阶指南"
```


# Events up

子组件通过 `emit` 发送事件通知父组件修改数据.

- **不能直接往回传**: 子组件不能直接改父组件里的变量.
- **发送信号**: 子组件如果想改变状态 (比如按钮被点了), 会通过 `emit` 进行请求, 父组件监听到后, 由父组件自己修改数据.
- **目的**: 确保父组件对自己的数据有最终决定权.

例如, 子组件中的修改书名按钮被点击:

```vue
<!-- ChildComponent.vue (子组件) -->
<template>
  <button @click="$emit('changeName', '新书名')">Edit Name</button>
</template>
```

```vue
<!-- Parent.vue (父组件) -->
<template>
  <!-- 父组件监听 @changeName 事件 -->
  <ChildComponent @changeName="handleUpdate" />
</template>

<script setup>
const handleUpdate = (newName) => {
  // 父组件在这里真正执行修改操作
  title.value = newName;
}
</script>
```

下列代码中:

```vue
<button @click="$emit('changeName', '新书名')">Edit Name</button>
```

`$emit` 的 `$` 表示这是vue自带的函数, 避免重名冲突.

`$emit` 的规则为:

```js
$emit('自定义事件名', 想要传给父组件的数据)
```


# 使用

在新版本中, 首先在子组件中定义:

```vue
<!-- 子组件 Child.vue -->
<script setup>
// 1. 先声明这个组件会发出哪些事件（这能让代码更清晰，也有利于 IDE 提示）
const emit = defineEmits(['update-title'])

function handleClick() {
  // 处理数据
  const newValue = '处理过的数据'
  // 2. 调用 emit 函数
  emit('update-title', newValue)
}
</script>

<template>
  <button @click="handleClick">点我执行函数再发送</button>
</template>
```

父组件的监听:

```vue
<!-- 父组件 Parent.vue -->
<template>
  <!-- 监听子组件发出的 'update-title' 事件 -->
  <Child @update-title="handleChildEvent" />
</template>

<script setup>
// 子组件传过来的 'Hello World' 会自动作为第一个参数传给函数
const handleChildEvent = (payload) => {
  console.log('接收到子组件的消息:', payload) // 输出: Hello World
}
</script>
```

使用时, 在父组件的 `<script setup>` 部分进行 `import`

```js
import ObjectProfile from './ObjectProfile.vue'
```

子组件中不需要 `export` , Vue会自动完成.

