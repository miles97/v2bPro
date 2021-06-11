#### 学习vue3的部分新特性

vue3中使用
const state = reactive({})定义一些常量，也就是vue2中使用的data函数。

生命周期函数基本不变 但是 为了使组合式 API 的功能和选项式 API 一样完整，我们还需要一种在 setup 中注册生命周期钩子的方法。

这要归功于 Vue 导出的几个新函数。组合式 API 上的生命周期钩子与选项式 API 的名称相同，但前缀为 on：即 mounted 看起来会像 onMounted

```js
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  onMounted(getUserRepositories) // 在 `mounted` 时调用 `getUserRepositories`

  return {
    repositories,
    getUserRepositories
  }
}
```

以及生命周期按需引入的关系 直接在
```
import { reactive, onMounted, ref, toRef
s } from 'vue'
```

Teleport 是什么呢？
Teleport 就像是哆啦 A 梦中的「任意门」，任意门的作用就是可以将人瞬间传送到另一个地方。有了这个认识，我们再来看一下为什么需要用到 Teleport 的特性呢，看一个小例子：
在子组件Header中使用到Dialog组件，我们实际开发中经常会在类似的情形下使用到 Dialog ，此时Dialog就被渲染到一层层子组件内部，处理嵌套组件的定位、z-index和样式都变得困难。
Dialog从用户感知的层面，应该是一个独立的组件，从 dom 结构应该完全剥离 Vue 顶层组件挂载的 DOM；同时还可以使用到 Vue 组件内的状态（data或者props）的值。简单来说就是,即希望继续在组件内部使用Dialog, 又希望渲染的 DOM 结构不嵌套在组件的 DOM 中。
此时就需要 Teleport 上场，我们可以用<Teleport>包裹Dialog, 此时就建立了一个传送门，可以将Dialog渲染的内容传送到任何指定的地方。
接下来就举个小例子，看看 Teleport 的使用方式

  Teleport 的使用
我们希望 Dialog 渲染的 dom 和顶层组件是兄弟节点关系, 在index.html文件中定义一个供挂载的元素:
<body>
  <div id="app"></div>
  <div id="dialog"></div>
</body>
复制代码
定义一个Dialog组件Dialog.vue, 留意 to 属性， 与上面的id选择器一致：
<template>
  <teleport to="#dialog">
    <div class="dialog">
      <div class="dialog_wrapper">
        <div class="dialog_header" v-if="title">
          <slot name="header">
            <span>{{ title }}</span>
          </slot>
        </div>
      </div>
      <div class="dialog_content">
        <slot></slot>
      </div>
      <div class="dialog_footer">
        <slot name="footer"></slot>
      </div>
    </div>
  </teleport>
</template>
复制代码
最后在一个子组件Header.vue中使用Dialog组件, 这里主要演示 Teleport 的使用，不相关的代码就省略了。header组件
<div class="header">
    ...
    <navbar />
    <Dialog v-if="dialogVisible"></Dialog>
</div>
...
复制代码
Dom 渲染效果如下：

图片. png
可以看到，我们使用 teleport 组件，通过 to 属性，指定该组件渲染的位置与 <div id="app"></div> 同级，也就是在 body 下，但是 Dialog 的状态 dialogVisible 又是完全由内部 Vue 组件控制.

