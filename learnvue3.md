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
  
Vite 是什么
Vite 是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：

一个开发服务器，它基于 原生 ES 模块 提供了 丰富的内建功能，如速度快到惊人的 模块热更新（HMR）。
一套构建指令，它使用 Rollup 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

Vite 意在提供开箱即用的配置，同时它的 插件 API 和 JavaScript API 带来了高度的可扩展性，并有完整的类型支持。
为什么使用 Vite
Vite 的开箱即用的配置非常让人省心，省去了很多繁琐复杂的 webpack 和 babel 配置流程，原生支持 TypeScript 的打包，调试的时候基本秒开项目，拥抱 ES Module 无需打包并且支持实时热更新。
或许有的人说不是由很多现成的脚手架可以直接使用了吗。Vue3 和 React 项目目前到时都有不错的基于 Vite
的脚手架工具，但是我面对的是一个比较冷门的 Web框架 的脚手架迁移。
使用 Vite 创建 OMI 脚手架
什么是 OMI
OMI-前端【跨框架】框架，既能以 Web Components 自定义标签形式赋能前端生态，如(react、vue、preact)，也可自成一套体系加速 Web 和 小程序前端开发。
OMI 可以说是一个可以用 JSX + WebComponent 的一个开发框架，加入 OMI 的周边生态开发中也学到了很多，包括源码上以及框架设计上的很多知识，与其主要贡献者也交流了很多次，也算是一种尝试加入开源社区的一种实践。
尝试使用 OMI-Vite 脚手架
安装与初始化项目
npm install -g omi-cli
omi init-vit my-app

基于 Vite 开发配置 OMI 脚手架
脚手架的主要功能就是（处理用户命令行输入，拉取项目模板，以及一部分自动化脚本）
Vite 作为一种新型前端构建工具，当然不仅仅适用于 Vue 的生态，目前以及支持了很多的模板。如(Vue,react,preact,lit-element,svelte)

1.使用 Vite 初始化项目模板
npm init @vitejs/app

这里直接选择 vanilla-ts （原生 TypeScript 项目）

敲击回车 Vite 项目初始化完成
2. 配置 OMI 项目开发环境
安装 OMI
npm install omi

安装开发依赖
Vite 支持各种主流 CSS 预处理器，只需要安装依赖开箱即用零配置
# .scss and .sass
npm install -D sass

# .less
npm install -D less

# .styl and .stylus
npm install -D stylus

Vite 也同样支持 PostCSS，如果项目包含有效的 PostCSS 配置 (任何受 postcss-load-config 支持的格式，例如 postcss.config.js)，它将会自动应用于所有已导入的 CSS。
Vite 原生支持各种文件类型读取打包，无需像 webpack 一样配置各种 loader ,Vite 可以使用插件进行扩展，这得益于 Rollup 优秀的插件接口设计和一部分 Vite 独有的额外选项。这意味着 Vite 用户可以利用 Rollup 插件的强大生态系统，同时根据需要也能够扩展开发服务器和 SSR 功能。
尝试编写 demo```
//packages/omi-cli/template/vite/src/main.tsx

import { WeElement, h, tag, render, } from 'omi'

interface HelloProps {
  name: string
}

@tag('hello-omi')
export default class extends WeElement<HelloProps> {

  render(props) {
    return (
      <div>Hello{props.name}</div>
		)
  }
}

render(<my-app name = 'Omi' > </hello-omi>, '#root')
```
3.配置 Vite.config.js
写完 Demo 运行一下

运行成功，正当我感觉万事大吉的时候

就奇怪 OMI 项目怎么会跑出 React 呢，由于 OMI 使用了 webcomponent + jsx 的形式，仔细一想是不是 JSX 的问题，于是查阅官方文档发现 .jsx 和 .tsx 文件同样开箱即用。JSX 的翻译同样是通过 ESBuild，默认为 React 16 形式。所以并没有执行 OMI 中对 JSX 的解析，需要在 Vite.config.js 中修改配置
```
于是照葫芦画瓢由于接口设计一致一通 Ctrl CV 猛如虎
export default {
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
}
```
至此整个项目已经可以正常运行
后面的就剩下完善一下 DEMO 的完整性了。
是的 Vite 使用起来就是这么简单，全部配置没超过十行。
使用 Vite (Rollup 进行库打包)
Vite 使用 Rollup 对项目进行打包，rollup打包出来的体积都比webpack略小一些，可读性也较高一些，可以看到的是像React、Vue等框架的构建工具使用的都是使用 rollup 进行打包，都使用 Vite 了没道理不尝试一下。





webpackrollup开发模式大小52.8KB19.46KB生产打包大小10.3KB7.66KB生产包gzip后大小4.1KB3.4KB
同样 Vite 也为应用打包提前做好了配置，但是我们这次要尝试的是库文件，所以需要对配置进行一些修改
构建过程可以通过多种构建配置选项来自定义，可以通过 vite.config.js 中的 build 选项进行配置```
const path = require('path')

module.exports = {
  build: {
    lib: {
      entry: path.resolve(__dirname, 'lib/index.ts'),
      name: 'cnmLib'
        },
    }
}

官方推荐在自己的库中的 package.json 这样配置
{
  "name": "my-lib",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.js",
  "module": "./dist/my-lib.es.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.es.js",
      "require": "./dist/my-lib.umd.js"
    }
  }
}
```
当需要构建你的库用于发布时，请使用 build.lib 配置项，请确保将你不想打包进你库中的依赖进行外部化，例如 vue 或 react：
// vite.config.js```
const path = require('path')

module.exports = {
  build: {
    lib: {
      entry: path.resolve(__dirname, 'lib/main.js'),
      name: 'MyLib'
    },
    rollupOptions: {
      // 请确保外部化那些你的库中不需要的依赖
      external: ['vue'],
      output: {
        // 在 UMD 构建模式下为这些外部化的依赖提供一个全局变量
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
}
```
运行 vite build 配合如上配置将会使用一套 Rollup 预设，为发行该库提供两种构建格式：es 和 umd（在 build.lib 中配置的）：```
$ vite build
building for production...
[write] my-lib.es.js 0.08kb, brotli: 0.07kb
[write] my-lib.umd.js 0.30kb, brotli: 0.16kb
```
  
当然你也可以通过底层的 rollup 配置文件去自定义打包配置，通过 build.rollupOptions 直接调整底层的 Rollup 选项：
编写 ```rollup.config.js
//rollup.config.js
export default RollupConfig = {
    input: 'src/lib/index.ts',
    output: {
        name: 'core',
        file: 'dist/lib/core.js',
        format: 'umd',
    },
    plugins: [],
}
```
vite.config.js 中引入```
import rollupConfig from './rollup.config'

module.exports = {
  build: {
    rollupOptions: rollupConfig
  }
}
```
rollup.config.js 需要根据项目需求自行配置
