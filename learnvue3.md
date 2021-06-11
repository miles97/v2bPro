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

最后在一个子组件Header.vue中使用Dialog组件, 这里主要演示 Teleport 的使用，不相关的代码就省略了。header组件
<div class="header">
    ...
    <navbar />
    <Dialog v-if="dialogVisible"></Dialog>
</div>
...

Dom 渲染效果如下：

图片. png
可以看到，我们使用 teleport 组件，通过 to 属性，指定该组件渲染的位置与 <div id="app"></div> 同级，也就是在 body 下，但是 Dialog 的状态 dialogVisible 又是完全由内部 Vue 组件控制.

v-model 升级
在使用 Vue 3 之前就了解到 v-model 发生了很大的变化， 使用过了之后才真正的 get 到这些变化， 我们先纵观一下发生了哪些变化， 然后再针对的说一下如何使用：

变更：在自定义组件上使用v-model时， 属性以及事件的默认名称变了
变更：v-bind的.sync修饰符在 Vue 3 中又被去掉了, 合并到了v-model里
新增：同一组件可以同时设置多个 v-model
新增：开发者可以自定义 v-model修饰符

有点懵？别着急，往下看 在 Vue2 中， 在组件上使用 v-model其实就相当于传递了value属性， 并触发了input事件：
<!-- Vue 2 -->
<search-input v-model="searchValue"><search-input>

<!-- 相当于 -->
<search-input :value="searchValue" @input="searchValue=$event"><search-input>

这时v-model只能绑定在组件的value属性上，那我们就不开心了， 我们就像给自己的组件用一个别的属性，并且我们不想通过触发input来更新值，在.sync出来之前，Vue 2 中这样实现：
// 子组件：searchInput.vue
export default {
    model:{
        prop: 'search',
        event:'change'
    }
}

修改后， searchInput 组件使用v-model就相当于这样：
<search-input v-model="searchValue"><search-input>
<!-- 相当于 -->
<search-input :search="searchValue" @change="searchValue=$event"><search-input>

但是在实际开发中，有些场景我们可能需要对一个 prop 进行 “双向绑定”， 这里以最常见的 modal 为例子：modal 挺合适属性双向绑定的，外部可以控制组件的visible显示或者隐藏，组件内部关闭可以控制 visible属性隐藏，同时 visible 属性同步传输到外部。组件内部， 当我们关闭modal时, 在子组件中以 update:PropName 模式触发事件：
this.$emit('update:visible', false)

然后在父组件中可以监听这个事件进行数据更新：
<modal :visible="isVisible" @update:visible="isVisible = $event"></modal>

此时我们也可以使用v-bind.sync来简化实现：
<modal :visible.sync="isVisible"></modal>

  
上面回顾了 Vue2 中v-model实现以及组件属性的双向绑定，那么在 Vue 3 中应该怎样实现的呢？
在 Vue3 中, 在自定义组件上使用v-model, 相当于传递一个modelValue 属性， 同时触发一个update:modelValue事件：
<modal v-model="isVisible"></modal>
<!-- 相当于 -->
<modal :modelValue="isVisible" @update:modelValue="isVisible = $event"></modal>

如果要绑定属性名， 只需要给v-model传递一个参数就行, 同时可以绑定多个v-model：
<modal v-model:visible="isVisible" v-model:content="content"></modal>

<!-- 相当于 -->
<modal
    :visible="isVisible"
    :content="content"
    @update:visible="isVisible"
    @update:content="content"
/>
不知道你有没有发现，这个写法完全没有.async什么事儿了， 所以啊，Vue 3 中又抛弃了.async写法， 统一使用v-model
  
  
  
  #### vite创建项目体验 以及打包相关
  
