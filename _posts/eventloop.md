---
title: 事件循环机制（Event Loop）
date: 2020-04-25 16:32:45
tags: JavaScript
---

事件循环机制（Event Loop）是全面了解javascript代码执行顺序绕不开的一个重要知识点。虽然许多人知道这个知识点非常重要，但是其实很少有人能够真正理解它。特别是在ES6正式支持Promise之后，对于新标准中事件循环的理解就变得更加重要了。这里我们不具体讲Event Loop（有很多文章说得已经足够好了），我们只是做个牵引——为什么需要Event loop和简单介绍。

<!-- more -->

在知道事件循环机制之前，我们得确信自己掌握了：

- 执行上下文（Execution context）
- 函数调用栈（call stack）
- 队列数据结构（queue）

希望你抛开已有javascript运行机制方面的相关知识，只要知道以上3点知识就好了。

先来看两个简单的例子，看看是否能够得出正确的

~~~js
//demo01
setTimeout(function() {
    console.log(1);
},0);
console.log(2);

for(var i = 0; i < 5; i++) {
    console.log(3);
}

console.log(4);
~~~

执行结果

~~~js
//demo02
console.log(1);

for(let i = 0; i< 5; i++){
    setTimeout(function() {
        console.log(i)
    },Math.floor(Math.random()*10+1)*100)

}

console.log(3)
~~~

（我们假设我们只知道以上3个知识点）很多人在运行之后可能会感到困惑，为什么即使设置了setTimeout的延迟时间为0，代码还是最后执行？再者定时器的时间（随机设定）优于遇到setTimeout的顺序，时间短的就为什么先执行？同常情况下，决定代码执行顺序的是函数调用栈（call stack），很显然这里的setTimeout中的执行顺序已经不是用call stack能够解释清除的了，这是为什么呢？答案是队列（queue）。

![img](https://user-gold-cdn.xitu.io/2018/6/22/16426a778da76817?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

""



**javascript的一个特点就是单线程，但是很多时候我们仍然需要在不同的时间去执行不同的任务，例如给元素添加点击事件，设置一个定时器，或者发起Ajax请求。因此需要一个异步机制**

**来达到这样的目的，事件循环机制也因此而来。**

那么问题来了浏览器是如何做到Event Loop这个机制的了？这就需要您自行去查找资料或是从分发挥您的想象力。

接下来，就简单介绍一下Event Loop:

每一个JavaScript程序都拥有唯一的事件循环，大多数代码的执行顺序是可以根据函数调用栈的规则执行的，而setTimeout/setInterval或者不同的事件绑定中的代码，则通过队列来执行。

setTimeout为任务源，或者任务分发器，由它们将不同的任务分发到不同的任务队列中去。每一个任务都有对应的任务队列。

任务队列又分为宏任务（macro-task）与微任务（micro-task）两种，在浏览器中（不说node），包括：

- macro-task：script(整体代码)、setTimeout/setInterval、I/O、UI rendering等。
- micro-task：Promise。

事件循环的顺序，决定了JavaScript代码的执行顺序。

它从macro-task中的script开始第一次循环。此时全局上下文进入函数调用栈（call stack），直到调用栈清空（只剩下全局上下文，你可以能为这是一个死循环，反正很蠢），在这个过程中，如果遇到任务分发器，就会将任务放入对应队列中去。

第一次循环时，macro-task中其实只有script，因此函数调用栈清空之后，会直接执行所有的micro-task。当所有可执行的微任务执行完毕之后，就表示第一次事件循环已经结束。

第二次循环会再次从macro-task开始执行。此时macro-task中的script队列中已经没有任务了，但是可能会有其它队列任务，而micro-task中暂时还没有任务。此时会先选择其中一个宏任务队列，例如setTimeout，将该队列中的所有任务全部执行完毕，然后再执行此过程中可能产生的微任务。微任务执行完毕之后，再回过头来执行其它宏任务队列中的任务。依次类推，直到所有宏任务队列中的任务都被执行一遍，并且清空了微任务，第二次循环就会结束。

如果在第二次循环过程中，产生了新的宏任务队列，或者之前宏任务队列中的任务暂时没有满足执行条件，例如延迟时间不够后者事件没有触发，那么将会继续同样的顺序重复循环