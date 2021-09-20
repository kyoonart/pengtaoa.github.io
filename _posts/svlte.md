---
title: 由一篇文章引申出的关于原生 JavaScript 在 createTextNode 和 innerText 操作的性能差异的探究
date: 2021-04-16 21:39:47
tags: JavaScript Dom
---

## 前言

昨天在浏览 推送的文章时，看到了一篇 **《[我对 Svelte 的看法](https://mp.weixin.qq.com/s/IB5Ht5Cp73ySsfqg608WPA)》** 的文章，看到其中一段代码示例，使用原生 `JavaScript` 实现一个不使用 `Virtual DOM` 的 Counter

<!-- more -->

```javascript
const target = document.querySelector('#app');

// state
let count = 0;

// view
const div = document.createElement('div');
const countText = document.createTextNode(`${count}`);
div.appendChild(countText);

const button1 = document.createElement('button');
const button1Text = document.createTextNode(`+`);
button1.appendChild(button1Text);

const button2 = document.createElement('button');
const button2Text = document.createTextNode(`-`);
button2.appendChild(button2Text);

target.appendChild(div);
target.appendChild(button1);
target.appendChild(button2);

// event
button1.addEventListener('click', () => {
  count += 1;
});
button2.addEventListener('click', () => {
  count -= 1;
});
```

发现示例中给相关的 DOM 插入 Text 使用的是 `document.createTextNode`，由于我在写代码的过程中基本没有使用过这种方式来创建 TextNode 并作为子节点插入到相关的 DOM 中，于是对这个方法有了一丝好奇。

通常情况下，我都会选择使用 `innerText` 的方式将 Text 插入到相关的 DOM 中，这就产生了一个问题：

- 在写代码的时候，到底是使用 `document.createTextNode` 创建 TextNode 并插入到相关的 DOM 中，还是直接使用 `innerText` 将 Text 插入更加高效？

## 思路

为了获得这个问题的答案，第一个想到的办法是使用 `Jest` 分别跑两段测试代码，然后查看两者之间的时间差。

但是仔细一想，`Jest` 是跑在 `Node` 环境下的，不是浏览器环境，也就没有 DOM 对象，只能通过 `JSDOM` 来模拟，这样的话就是运行纯 `JavaScript` 了，少了 `JavaScript` 通过 `Web IDL` 修改 DOM 对象的过程，也就没办法测出这两种方式的差异。

关于这个知识点，可以看一下这份知乎回答[前端为什么操作 DOM 是最耗性能的呢？ - justjavac 的回答 - 知乎](https://www.zhihu.com/question/324992717/answer/707044362)

不过嘛，既然是测试，跑一跑也无妨，验证一下是不是真的如我想的那样无法测出**非 DOM** 操作和 **DOM 操作**之间的差异。`JSDOM`

### `document.title` & `document.xxx`

先上两段代码，来验证一下在 `Node` 环境下是不是真的无法测出**非 DOM** 操作和 **DOM 操作**之间的差异。

jsdom/dom.js

```javascript
const {JSDOM} = require('jsdom');

const {window} = new JSDOM(``);

const {document} = window;

console.time();
for (let i = 0; i <= 1e4; i++) {
  document.title = i;
}
console.timeEnd();
```

jsdom/non-dom.js

```javascript
const {JSDOM} = require('jsdom');

const {window} = new JSDOM(``);

const {document} = window;

console.time();
for (let i = 0; i <= 1e4; i++) {
  document.xxx = i;
}
console.timeEnd();
```

运行之后发现，事实并非我们想象的那样：

jsdom/dom.js

```shell
$ node jsdom/dom.js

default: 195.461ms
```

jsdom/non-dom.js

```shell
$ node jsdom/non-dom
default: 0.281ms
```

问题来了，为什么即使是在 `Node` 环境下也有如此大的差异？

Debug 了一轮之后发现，对 `document.title` 做修改的时候，会触发 `setter`：

![](http://cdn.jack-wjq.cn/PicGo/image-20210330113734010.png)

而对 `document.xxx` 做修改的时候仅仅只是一个赋值操作：

![](http://cdn.jack-wjq.cn/PicGo/image-20210330114208332.png)

这就出现区别了，`document` 对象是使用 `Object.defineProperty` 包装过部分属性的，其中 `title` 就重写了它的 `getter/setter`，因此在修改 `document.title` 的时候耗时明显增加。

不过这是在测试修改 `document.title` 和 `document.xxx` 的差异，跑题了！

现在回到测试 `document.createTextNode` 和 `innerText` 这两者的差异上。

### `createTextNode` & `innerText`

createTextNode.js

```javascript
const {JSDOM} = require('jsdom');

const {window} = new JSDOM(`
  <div id="root"></div>
`);

const {document} = window;

const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e4; i++) {
  const text = document.createTextNode(i);
  root.append(text);
}
console.timeEnd();
```

innerText

```javascript
const {JSDOM} = require('jsdom');

const {window} = new JSDOM(`
  <div id="root"></div>
`);

const {document} = window;

const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e4; i++) {
  root.innerText += i;
}
console.timeEnd();
```

结果出人意料：

```shell
$ node jsdom/createTextNode

default: 103.349ms

$ node jsdom/innerText

default: 0.459ms
```

非常意外！于是我又 Debug 了一轮，发现 `JSDOM` 提供的 `getElementById` 的方法返回的 `HTMLElement` 并没有 `innerText` 这个属性。

与上面修改 `document.xxx` 的情况一样，修改 `innerText` 变成了一个简单的赋值操作。

## Chrome

### V1

### 既然在 `Node` 环境下没办法完成这个测试，于是我将试验场更换到了 Chrome，让我们看一下在 Chrome 中运行会有什么差异。

于是我创建了两个页面来进行测试：

createTextNode.html

```javascript
<div id="root"></div>
<script>
  const root = document.getElementById('root');

  console.time();
  for (let i = 0; i <= 1e4; i++) {
    const text = document.createTextNode(i);
    root.appendChild(text);
  }
  console.timeEnd();
</script>
```

innerText.html

```javascript
<div id="root"></div>
<script>
  const root = document.getElementById('root');

  console.time();
  for (let i = 0; i <= 1e4; i++) {
    root.innerText += i;
  }
  console.timeEnd();
</script>
```

结果出人意料，差距实在太大了：

```shell
 default: 3.133056640625 ms
 
 default: 11603.7099609375 ms
```

这样看来，`createTextNode` 的性能显然比修改 `innerText` 好太多了。

不应该会有这么大的差距啊！是什么问题呢？

回到 `JavaScript` 代码，对比之后发现，修改 `innerText` 为了实现和 `createTextNode` 一样的行为，有一个 `+=` 操作。

难道是这里耗时严重？

思考之后，决定重新设计这个实验，去掉 `+=` 的操作，同时要保证最终的实现效果一致。

V2

createTextNode.html

```js
<div id="root"></div>
<script>
  const root = document.getElementById('root');

  console.time();
  for (let i = 0; i <= 1e4; i++) {
    const child = document.createElement('div');
    const text = document.createTextNode(i);
    child.appendChild(text);
    root.appendChild(child);
  }
  console.timeEnd();
</script>
```

innerText.html

```js
<div id="root"></div>
<script>
  const root = document.getElementById('root');

  console.time();
  for (let i = 0; i <= 1e4; i++) {
    const child = document.createElement('div');
    child.innerText = i;
    root.appendChild(child);
  }
  console.timeEnd();
</script>
```

实验改进之后，应该算严格控制了两个实验行为一致了，去掉了 `+=` 操作，现在结果就非常接近了：

```js
createTextNode.html

 default: 6.649169921875 ms

innerText.html

 default: 6.679931640625 ms
```

为了确保这个结果是准确的，我们将循环次数放大一个数量级，再测试一次。

循环次数放大一个数量级之后，结果是这样的：

```js
createTextNode.html

 default: 54.6650390625 ms

Copy

innerText.html

 default: 54.66015625 ms
```

依然非常接近，基本没有差别。

是不是就能确定这两种不同的写法性能上没区别呢？

事实上改进后的实验，可能还存在问题，问题出在哪呢？

V3

其实问题出在了循环体中的代码顺序上：

CreateTextNode.js

```js
const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e5; i++) {
  const child = document.createElement('div');
  const text = document.createTextNode(i);
  child.appendChild(text);
  root.appendChild(child);
}
console.timeEnd();
```

innerText.js

```js
const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e5; i++) {
  const child = document.createElement('div');
  child.innerText = i;
  root.appendChild(child);
}
console.timeEnd();
```

为什么说问题出在了循环体中的代码顺序上呢？

因为在 `child` 被插入到 `root` 之前，它还是一个 `JavaScript` 对象，对 `JavaScript` 对象的属性做修改的速度是非常快的。

所以我们又修改了一下，把 `child` 插入 `root` 的代码提前，先插入到 `root` 中，再修改 Text。

createTextNode.js

```js
const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e5; i++) {
  const child = document.createElement('div');
  root.appendChild(child);
  const text = document.createTextNode(i);
  child.appendChild(text);
}
console.timeEnd();
```

innerText.js

```js
const root = document.getElementById('root');

console.time();
for (let i = 0; i <= 1e5; i++) {
  const child = document.createElement('div');
  root.appendChild(child);
  const text = document.createTextNode(i);
  child.appendChild(text);
}
console.timeEnd();
```

再来看看结果

```shell
createTextNode.html
 default: 57.946044921875 ms
innerText.html
 default: 56.62109375 ms
```

到这里我们应该能够确定，这两种方式插入 Text 在性能上没有区别了。

## 总结

因为一次技术文章的阅读引出这么一些内容，然后测试了一下，也不确定测试方法对不对，如果有跟了解的大佬可以指点一下~