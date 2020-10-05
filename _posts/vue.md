---
title: vue内部运行机制
date: 2020-07-25 12:26:18
tags: vue
---

一张图了解全貌vue运行机制的全貌

<!-- more --> 

![](https://s1.ax1x.com/2020/10/01/0MPliF.jpg)

在 `new Vue()` 之后。 Vue 会调用 `_init` 函数进行初始化，也就是这里的 `init` 过程，它会初始化生命周期、事件、 props、 methods、 data、 computed 与 watch 等。其中最重要的是通过 `Object.defineProperty` 设置 `setter` 与 `getter` 函数，用来实现「**响应式**」以及「**依赖收集**」

初始化之后调用 `$mount` 会挂载组件，如果是运行时编译，即不存在 render function 但是存在 template 的情况，需要进行「**编译**」步骤。



#### 关于两种编译模式

  Vue 中提供了两种编译模式：

  (1)完整版： 运行时 + 编译器

   1.1 在页面中直接通过 script 标签引入的 vue.js 文件就是 完整版 的

   既可以使用 render 函数，也可以使用 template 模板

  (2)只包含运行时版本

   2.1 我们自己手动通过 webpack 配置的打包环境中，通过 import Vue from ‘vue’

​     导入的Vue就是 运行版本 的，也就是：`vue.runtime.esm.js`

   只能使用 render 函数来渲染组件

  以上两个版本的区别：完整版中既包含了运行时 又包含了 编译器，编译器就是用来解析

  Vue模板的（ 可以理解为就是 Vue实例中 template 配置项 ）

  如果在 运行时版本 中，是无法使用 template 模板内容的，因为 运行时版本 中没有编译器

  也就无法编译 template 模板内容。此时，应该使用 render 函数来渲染组件内容



### 编译

compile编译可以分成 `parse`、`optimize` 与 `generate` 三个阶段，最终需要得到 render function

![](https://s1.ax1x.com/2020/10/01/0MPJMR.jpg)

### parse

`parse` 会用正则等方式解析 template 模板中的指令、class、style等数据，形成AST。

### optimize

`optimize` 的主要作用是标记 static 静态节点，这是 Vue 在编译过程中的一处优化，后面当 `update` 更新界面时，会有一个 `patch` 的过程， diff 算法会直接跳过静态节点，从而减少了比较的过程，优化了 `patch` 的性能。

### generate

`generate` 是将 AST 转化成 render function 字符串的过程，得到结果是 render 的字符串以及 staticRenderFns 字符串。

在经历过 `parse`、`optimize` 与 `generate` 这三个阶段以后，组件中就会存在渲染 VNode 所需的 render function 了。

### 响应式

Vue.js 响应式核心部分

![](https://s1.ax1x.com/2020/10/01/0MPUZ6.jpg)

在 `init` 的时候通过 `Object.defineProperty` 进行了绑定，它使得当被设置的对象被读取的时候会执行 `getter` 函数，而在当被赋值的时候会执行 `setter` 函数。

当 render function 被渲染的时候，因为会读取所需对象的值，所以会触发 `getter` 函数进行「**依赖收集**」，「**依赖收集**」的目的是将观察者 Watcher 对象存放到当前闭包中的订阅者 Dep 的 subs 中。形成如下所示的如下这样的一个关系。



![](https://s1.ax1x.com/2020/10/01/0MPdIO.jpg)

在修改对象的值的时候，会触发对应的 `setter`， `setter` 通知之前「**依赖收集**」得到的 Dep 中的每一个 Watcher，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 `update` 来更新视图，当然这中间还有一个 `patch` 的过程以及使用队列来异步更新的策略.

**一个对象属性对应一个dep，一个dep对应多个watcher(一个对象属性可能再多个标签使用，那么就会有对应多个watcher，这些watcher都会放入到这个对象属性唯一对应的dep中)，这是Vue1.0的实现，但数据过大时，就会有很多个watcher，就会出现性能问题；所以在Vue2.0中引入的VDOM，给每个vue组件绑定一个watcher，这个组件上的数据的dep中都包含有该watcher，当该组件数据发生变化时，就会通知watcher触发update方法，生成VDOM，和旧的VDOM进行比较，更新改变的部分，极大的减少了watcher的数量，优化了性能；（所以，在Vue2.0中是一个组件对应一个watcher）**

### Virtual DOM 虚拟Dom

render function 会被转化成 VNode 节点。Virtual DOM 其实就是一棵以 JavaScript 对象（ VNode 节点）作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。

比如说下面这样一个例子：

~~~js
{
    tag: 'div',                 /*说明这是一个div标签*/
    children: [                 /*存放该标签的子节点*/
        {
            tag: 'a',           /*说明这是一个a标签*/
            text: 'click me'    /*标签的内容*/
        }
    ]
}
~~~

渲染后可以得到这样的节点

~~~html
<div>
    <a>click me</a>
</div>
~~~

这只是一个简单的例子，实际上的节点有更多的属性来标志节点，比如 isStatic （代表是否为静态节点）、 isComment （代表是否为注释节点）等

### 更新视图

![](https://s1.ax1x.com/2020/10/01/0MPBJe.jpg)

在修改一个对象值的时候，会通过 `setter -> Watcher -> update` 的流程来修改对应的视图，那么最终是如何更新视图的呢？

当数据变化后，执行 render function 就可以得到一个新的 VNode 节点，我们如果想要得到新的视图，最简单粗暴的方法就是直接解析这个新的 VNode 节点，然后用 `innerHTML` 直接全部渲染到真实 DOM 中。但是其实我们只对其中的一小块内容进行了修改，这样做似乎有些「**浪费**」。

那么我们为什么不能只修改那些「改变了的地方」呢？这个时候就要介绍我们的「**`patch`**」了。我们会将新的 VNode 与旧的 VNode 一起传入 `patch` 进行比较，经过 diff 算法得出它们的「**差异**」。最后我们只需要将这些「**差异**」的对应 DOM 进行修改即可。下一篇再分析分析patch过程到底是如何进行的~

