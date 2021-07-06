### 浏览器相关知识点

浏览器的主要组件包括：

用户界面 (User Interface) － 包括地址栏、后退/前进按钮、书签目录等，也就是你所看到的除了用来显示你所请求页面的主窗口之外的其他部分。
浏览器引擎 (Browser engine) － 用来查询及操作渲染引擎的接口。
渲染引擎 (Rendering engine) － 负责解析用户请求的内容(如HTML或XML，渲染引擎会解析HTML或XML，以及相关CSS，然后返回解析后的内容)
网络(Networking)－ 用来完成网络调用，例如http请求，它具有平台无关的接口，可以在不同平台上工作。
UI后端(UI backend) － 用来绘制类似组合选择框及对话框等基本组件，具有不特定于某个平台的通用接口，底层使用操作系统的用户接口。
JS解释器(JavaScript interpreter) － 用来解释执行JS代码。
数据存储 (Data storage) － 属于持久层，浏览器需要在硬盘中保存类似cookie的各种数据，HTML5定义了web database技术，这是一种轻量级完整的客户端存储技术

#### 输入网址到页面呈现

不考虑重定向的话，输入网址到页面呈现大致有下面五步：

- DNS 查询

- TCP 连接

- HTTP 请求即响应

- 服务器响应

- 客户端渲染

  

### 关键渲染路径

当浏览器接收到服务器接返回的一个HTML页面，到屏幕上渲染出来要经过很多个步骤。浏览器完成这一系列的运行(即渲染出来)，我们称之为“关键渲染路径”（Critical Rendering Path） 下面是渲染引擎在取得内容之后的基本流程：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5dcec01f0f424e78ba217cb3845b3181~tplv-k3u1fbpfcp-zoom-1.image)

接下来我们认识几个概念：

- DOMTree：浏览器将HTML解析成树形的数据结构。
- CSSRuleTree：浏览器将CSS解析成树形的数据结构。CSSOM是CSS Object Model的缩写,它和DOM类似，但是只针对CSS而不是HTML。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70cf50162da8422887eb9501964a961a~tplv-k3u1fbpfcp-zoom-1.image)

- RenderTree: DOM和CSSOM合并后生成Render Tree。
- layout: 有了Render Tree，浏览器已经能知道网页中有哪些节点、各个节点的CSS定义以及他们的从属关系，从而去计算出每个节点在屏幕中的位置。
- painting: 按照算出来的规则，通过显卡，把内容画到屏幕上。
- reflow（回流）：当浏览器发现某个部分发生了点变化影响了布局，需要倒回去重新渲染，内行称这个回退的过程叫 reflow。reflow 会从 这个 root frame 开始递归往下，依次计算所有的结点几何尺寸和位置。reflow 几乎是无法避免的。现在界面上流行的一些效果，比如树状目录的折叠、展开（实质上是元素的显 示与隐藏）等，都将引起浏览器的 reflow。鼠标滑过、点击……只要这些行为引起了页面上某些元素的占位面积、定位方式、边距等属性的变化，都会引起它内部、周围甚至整个页面的重新渲 染。通常我们都无法预估浏览器到底会 reflow 哪一部分的代码，它们都彼此相互影响着。
- repaint（重绘）：改变某个元素的背景色、文字颜色、边框颜色等等不影响它周围或内部布局的属性时，屏幕的一部分要重画，但是元素的几何尺寸没有变。





