### 用户交互实践

目前PC端接触的比较多，相应的开发任务也基本都是pc端的任务，基本的交互优化的细节有

1.禁止选中

由于很多表单元素默认的选中方式都可以复制，所以很多时候需要user-select：none去禁用选中效果，特别是一些需要点击的元素。防止错误操作



2.指针样式

特别的，很多时候指针的样式也是需要优化的方面，目前来说单页面应用的表单概念较轻，所以针对于指针的样式有时候需要特定的加载成手指，或者普通的点击cursor:point;



3.改变浏览器的默认滚动样式

页面内可以通过scope进行局部限制，否则就是全局加载样式替换浏览器默认样式。优化了美感，但是如果有横向移动的页面，滚动条可能难以拖拽。

```js
::-webkit-scrollbar {
    width: 4px;
    height: 4px;
}

::-webkit-scrollbar-thumb {
    border-radius: 5px;
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    background: rgba(0, 0, 0, 0.2);
}

::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 0;
    background: rgba(0, 0, 0, 0.1);
}
```



4.使用less或者styl等css与渲染器进行简单的css开发

规范的定义一些常用的全局颜色，常用的改变字体颜色以及宽高的方法。方便构建样式文件代码

```js
// //宽高
.wh(@width, @height){
  width: @width;
  height: @height;
}

// //字体大小，颜色
.sc(@size, @color){
  font-size: @size;
  color: @color;
}
@import './base.less';
@primary: #1baeae; // 主题色
@orange: #FF6B01; 
@bc: #F7F7F7;
@fc:#fff;
```





