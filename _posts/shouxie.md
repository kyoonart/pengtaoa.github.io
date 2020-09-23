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

