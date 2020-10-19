---
title: 记录一些常见的手写代码题目
date: 2020-04-13 16:30:10
tags: JavaScript
---

### 记录一些常见的手写代码题目

### 实现数据类型的判断

~~~js
function _type(obj) {
    let res = Object.prototype.toString.call(obj);
    let arr = res.substr(8).split('');
    let x = arr.length - 1;
    arr.splice(x, 1);
    return arr.join('')
}
//  更加简洁版本
var  _type = function(obj) {
    return Object.prototype.toString.call(obj).slice(8, -1);
}
// 关键点 利用了 Object.prototype.toString()这个方法(数据类型内部有个[[call]]属性，里面存储了每种数据类型，这个属性只有Object.prototype.toString()能访问到)
~~~

<!-- more -->

### 实现Promise.All

解释：只有当所有的promise都是resolve时才算成功否则就是reject，如果参数中 `promise` 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 `promise` 的结果。

~~~js
function isPromise(obj) {
    return !!obj && (typeof obj === 'object' || typeof obj === 'function') &&
        typeof obj.then === 'function';
}
function _PromiseAll(arr) {
    let result = [];
    return new Promise((resolve, reject) => {
        arr.forEach((item, index) => {
            if (isPromise(item)) {
                item.then((data) => {
                    result[index] = data
                    if (result.length === arr.length) {
                        resolve(result)
                    }
                }, reject)
            }
        })
    })
}
// 主要是要知道promise.all做了什么 顺便还了解了promsie.race
~~~

### 实现Promise.race

解释：竞速 当某个promise执行完成resolve那么就是resolve

`Promise.race(iterable)` 方法返回一个 `promise`，一旦迭代器中的某个 `promise` 解决或拒绝，返回的 `promise` 就会解决或拒绝。

```js
function isPromise(obj) {
    return !!obj && (typeof obj === 'object' || typeof obj === 'function') &&
        typeof obj.then === 'function';
}
function _promiseRace(promises){
     if(!promises.length) return
   return new Promise((resolve,reject)=>{
       promises.forEach(item=>{
           if(isPromise){
               item.then(data=>resolve(data))
           }
       },reject)
   })
       
   
}
```



### 实现发布订阅

~~~js
class Event {
    constructor() {
        this.callbacks = {}
    }
    $on(name, fn) {
        if ((this.callbacks[name]) || (this.callbacks[name] = [])) {
            this.callbacks[name].push(fn)
        }
    }
    $emit(name, args) {
        let cbs = this.callbacks[name]
        if (cbs) {
            cbs.forEach(v => v.call(this, args))
        }

    }
    $remove(name) {
        this.callbacks[name] = null
    }
}
// test
let event = new Event()
event.$on('hook', function(args) {
    console.log('hook', args);

})
event.$on('hook2', function(args) {
    console.log('hook2', args);

})
event.$remove('hook')
event.$emit('hook', '233')
event.$emit('hook2', '24444')
// 了解vue的都应该了解发布者订阅者模式
~~~

