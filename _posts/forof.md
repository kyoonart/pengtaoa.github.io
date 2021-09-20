---
title: 探究Forof实现原理
date: 2020-11-22 23:22:53
tags: JavaScript
---

探究一下for of实现原理

for of 是我们经常使用的一个属性，它可以非常方便的遍历数组，除此之外，它还可以用来遍历类数组对象，Set 和 Map 等，那么问题就来了，为什么它可以遍历 Map 却不能遍历普通的对象呢？那就来探索一下 for of 底层的秘密。

首先，我们知道 for of 是用来遍历可迭代对象的，那么如何判断一个对象是否是可迭代对象呢？ js 引擎是通过判断对象是否具有 `Symbol.iterator` 来判断的，如下：

```js
[][Symbol.iterator] // [Function: values]
{}[Symbol.iterator] // undefined
```

我们看到数组具有 `Symbol.iterator` 属性，并返回了一个函数，而对象返回了 undefined 说明没有该属性，所以普通的对象不是可迭代对象。那么 for of 底层是否是通过`Symbol.iterator` 来实现遍历的呢？我们来验证一下，代码如下：

<!-- more -->

```js
const obj = {
  0: 'a',
  1: 'b',
  length: 2,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
}

for (const item of obj) {
  console.log(item) // a b
}

```

答案是肯定，这里解释下什么是迭代器，其实所谓的迭代器就是具有 next() 方法的对象，每次调用 next() 都会返回一个结果对象，该结果对象有两个属性，value 表示当前的值，done 表示遍历是否结束。知道了迭代器的特点，我们就可以模拟一个 forof 方法，代码如下：

```js
let arr = [5, 6, 7, 8, 9]

function forof(arr, cb) {
  let iterator = arr[Symbol.iterator]()

  let result = iterator.next()
  while (!result.done) {
    cb(result.value)
    result = iterator.next()
  }
}

forof(arr, function (item) {
  console.log(item)
})
```

既然知道了 for of 的实现原理，那么我们是不是可以将普通的对象改造成可迭代对象？答案是肯定的，代码如下：

```js
function createIterator(obj) {
  let arr = Object.entries(obj)
  let i = 0
  return {
    next() {
      let done = i >= arr.length ? true : false
      let value = !done ? arr[i++] : undefined
      return {
        done,
        value
      }
    }
  }
}

const obj = {
  name: 'iwen',
  age: 18
}

obj[Symbol.iterator] = function () {
  return createIterator(obj)
}
for (const [key, value] of obj) {
  console.log(key, value)  // name iwen  age 18
}

```

