---
title: react生命周期总结
date: 2021-07-16 20:39:47
tags: JavaScript React
---

### 一、React15 生命周期介绍

- React 15生命周期方法汇总



```js
constructor() // 构造函数
componentWillReceiveProps() // 父组件状态属性更新触发
shouldComponentUpdate() // 组件更新时调用，在此可拦截更新
componentWillMount() // 初始化渲染时调用（挂载前调用）
componentWillUpdate() // 组件更新时调用
componentDidUpdate() // 组件更新后调用
componentDidMount() // 初始化渲染时调用（挂载后调用）
render() // 生成组件虚拟Dom
componentWillUnmount() // 组件卸载时调用

```

- React 15 各生命周期阶段介绍

<!-- more --> 

###### 1. Mounting 阶段：组件的初始化渲染（挂载）

`组件挂载->constructor()->componentWillMount()->render()->componentDidMount()`

首先调用 constructor 构造函数，然后依次调用 componentWillMount、render、componentDidMount，注意，componentWillMount是在render方法前触发的，所以在componentWillMount 方法中的一些初始化及操作DOM等操作是伴随一些风险的。

###### 2.Updating 阶段：组件的更新

`组件更新->componentWillReceiveProps()->shouldComponentUpdate（）->componentWillUpdate（）-> render（）->componentDidUpdate()`

可以看出，父组件触发的更新和组件自身触发的更新多出了 componentWillRecriveProps 这样一个方法。
componentWillRecriveProps 这个方法接受一个参数 nextProps，表示接收到新Props的内容。其中，componentWillRecriveProps 的触发条件是由父组件的更新触发的。之所以这样说，是因为很多小伙伴（包括我）之前一直认为，只有当父组件传递给子组件本身的props发生的变化才会触发子组件本身的 componentWillRecriveProps，这个结论实际上是不严谨的。
shouldComponentUpdate 方法主要是用来决定React组件是否还需要执行之后的生命周期方法，进而决定是否对组件进行re-render（重渲染）。频繁的render可能会产生性能问题，而shouldComponentUpdate 方法可阻止掉一部分不必要的render。



###### 3.Unmounting 阶段：组件的卸载

卸载阶段非常简单，卸载时只会调用 componentWillUnmount 方法，开发者可以再此做一些监听的清除和重置操作。

### 二、React16 生命周期介绍

- React 16生命周期方法汇总

```js
constructor() // 构造函数
getDerivedStateFromProps() // 组件初始化和更新时调用
shouldComponentUpdate() // 组件更新时调用，在此可拦截更新
render() // 生成虚拟Dom
getSnapshotBeforeUpdate() // 组件更新时调用
componentDidMount() // 组件初始化时调用（挂载后调用）
componentDidUpdate() // 组件更新后调用
componentWillUnmount() // 组件卸载时调用

```

- React 16 各生命周期阶段介绍

###### 1. Mounting 阶段：组件的初始化渲染（挂载）

`constructor() -> getDerivedStateFromProps() ->render()->componentDidMount()`

大体流程与React 15基本一致，但是其中的 componentWillMount 被更换成了 getDerivedStateFromProps。
getDerivedStateFromProps是一个静态方法，静态方法不依赖组件实例而存在，所以 getDerivedStateFromProps 中访问不到this。并且getDerivedStateFromProps 并不是 componentWillMount 的替代，他们的区别我们后面会讲到，我们先来说一下 getDerivedStateFromProps 存在的作用。
getDerivedStateFromProps有且仅有一个用途：使用props 来 派生/更新 state。
getDerivedStateFromProps 可以接收两个参数：props 和 state。其中props是来自父组件的props，state是组件自身的state。
getDerivedStateFromProps 需要一个对象格式的返回值。getDerivedStateFromProps 在组件初始化和更新时都会调用，其还会返回一个state对象，这个对象会融合到组件的state中，也就是会更新state。如果我们在这个方法中没有返回，在开发环境下，React 16会报一个警告，所以如果我们不需要在 getDerivedStateFromProps 中执行相应代码的话，最好不要将这个方法定义到我们的组件中，如果非要定义，记得一定要返回一个null。
值得注意的是，React 16.4 后对 getDerivedStateFromProps 做了微调。在16.4（包括16.4）以后的版本中，组件任何的更新流程都会触发 getDerivedStateFromProps 方法，而在16.4以前，只有父组件的更新会触发该生命周期。

###### 2.Updating 阶段：组件的更新

`组件更新，由父组件更新->getDerivedStateFromProps->shouldComponentUpdate()->render()->getSnapshotBeforeUpdate()->componentDidUpdate()	 `

React 16更新阶段的流程与15就大相径庭了，其中有两个生命周期被更改，并且执行顺序发生了一些变化。
首先 componentWillReceiveProps 被替换成了getDerivedStateFromProps。关于 getDerivedStateFromProps，我们在挂载阶段已经详细讲过，这里不再赘述。关于二者的区别，我们也后面来讲。
其次是将render前的 componentWillUpdate 移除了，在render后新增了一个 getSnapshotBeforeUpdate。
getSnapshotBeforeUpdate 执行在render方法之后，真实Dom更新之前。在这个阶段我们可以同时获取到更新前的真实Dom和更新前后的state和props信息。
getSnapshotBeforeUpdate 的返回值会作为第三个参数给到componentDidUpdate。
getSnapshotBeforeUpdate的设计初衷就是为了和componentDidUpdate 一起涵盖过时的 componentWillUpdate方法的所有用例。getSnapshotBeforeUpdate要想发挥作用，离不开 componentDidUpdate 的配合。

###### 3.Unmounting 阶段：组件的卸载

3.Unmounting 阶段：组件的卸载

### 三、React15 和 React16 生命周期区别对比

- getDerivedStateFromProps 和 componentWillMount

`为何React 16要替换掉 componentWillMount 呢？
其实 componentWillMount 的存在不仅鸡肋，而且风险极高。很多开发者（也包括我）在React15的 componentWillMount 中或多或少都栽过跟头，喜欢在其中去写一些异步代码和初始化操作，从而造成组件数据或真实Dom渲染上的一系列问题。总的来说，getDerivedStateFromProps并不是想要替代 componentWillMount，因为 componentWillMount 就应该被废弃。`

- getDerivedStateFromProps 和 componentWillReceiveProps

`为何React 16要替换掉 componentWillReceiveProps 呢？
首先，getDerivedStateFromProps 是作为一个试图替代 componentWillReceiveProps 的API出现的，但是，它又不完全等于 componentWillReceiveProps。我们前面说过，getDerivedStateFromProps有且仅有一个用途：使用props 来 派生/更新 state。所以，getDerivedStateFromProps 非常专注，它的用途非常简单。而 componentWillReceiveProps 中可以做更多的事，这样说来，componentWillReceiveProps比getDerivedStateFromProps功能更强大啊，为何要被替换掉呢？其实，一个API并非越庞大越复杂才越优秀。getDerivedStateFromProps正是在componentWillReceiveProps上做了合理的减法。从getDerivedStateFromProps 被定义成静态方法就可以看出React团队是为了通过对生命周期方法的限制来帮助开发者们避免掉一些不必要的坑。React 16正是在强制推行“只用getDerivedStateFromProps来完成props到state的映射”，从根源上帮开发者避免不合理的编程方式以及对生命周期的滥用。`

- getSnapshotBeforeUpdate 和 componentWillUpdate

`我们前面说过，getSnapshotBeforeUpdate的设计初衷就是为了和componentDidUpdate 一起涵盖过时的 componentWillUpdate方法的所有用例。所以，他们的区别其实与前面 getDerivedStateFromProps 和 componentWillReceiveProps 的区别类似。至于 componentWillUpdate 为何非死不可呢，关键还是因为它挡了新的 Fiber 架构的路`

- Fiber架构设计原理

`Fiber是React16对React核心算法的一次重写，Fiber会使原本同步的渲染过程变成异步的。
在React16之前，每当我们触发一次组件的更新，React都会构建一颗新的虚拟Dom树，通过与上一次的虚拟Dom树进行diff，实现对Dom的定向更新。这个过程，是一个递归的过程。
同步渲染的递归调用栈是非常深的。这个漫长且不可打断的更新过程，将会带来用户体验层面的巨大风险：同步渲染一旦开始，变回牢牢抓住主线程不放，直至递归彻底完成。在这个过程中，浏览器没有办法处理任何渲染之外的事情，会进入一种无法处理用户交互的状态。因此若渲染时间稍微长一点，页面就会面临卡顿甚至卡死的风险。
而Fiber正好能够解决掉这个风险：Fiber会将一个大的更新任务拆解成许多小任务。每当执行完一个小任务时，渲染线程都会把主线程交回去，看看有没有优先级更高的工作要处理，确保不会出现其他任务被“饿死”的情况，进而避免同步渲染带来的卡顿。
Fiber架构的重要特征就是可以被打断的异步渲染模式。但这个“打断”是有原则的。根据能否被打断这一标准，React16 的生命周期被划分为了render和commit两个阶段，而commit又被细分为了pre-commit和commit。如图：`

![](https://pic4.zhimg.com/v2-8c9f2b2eebc3449da805e8bd0deced47_r.jpg)

**render 阶段：纯净且没有副作用，可能会被React暂停、终止或重新启动。
pre-commit 阶段：可以读取Dom。
commit 阶段：可以使用Dom，运行副作用，安排更新。**
总的来说，render阶段在执行过程中允许被打断，而commit阶段则总是同步执行的`

- React16 中生命周期废旧立新的思考

`React 16废弃了三个生命周期函数，分别是：componentWillMount、componentWillUpdate以及componentWillReceiveProps。为何要废弃掉它们呢？
在Fiber机制下，render阶段是允许暂停、终止和重启的。这就导致render阶段的生命周期都是有可能被重复执行的。而这几个被废弃的生命周API常年被滥用，它们在重复执行的过程中都存在着不容小觑的风险。比如在这些生命周期中使用setState、fetch发起异步请求、操作真实Dom等等。而这些操作完全可以转移到其他生命周期中去做。而在Fiber带来的异步渲染机制下，这些生命周期都有可能被重复执行，所以可能会导致非常严重的bug。即使没有开启异步渲染，React15中也可能导致一些严重的问题，比如在componentWillReceiveProps 和 componentWillUpdate里滥用setState导致重复渲染死循环。`