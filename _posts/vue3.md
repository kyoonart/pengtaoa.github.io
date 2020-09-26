---
title: Vue3.0  Composition API 
date: 2020-08-26 13:17:13
tags: vue
---

在小米实习的时候使用到的技术栈、发现小米早在半年前就已经开始使用了、所以打算简单记录一下，不得不说小米前端所使用的的技术栈还是比较新的！

Vue3.0  Composition API  

<!-- more -->

```java
vue -V
vue add vue-next 升级vue版本
npm package
或者暂时不想升级技术栈 引用这个npm包也可以
+ yarn add @vue/composition-api
```

 Composition API

+ 什么是 Componsition API
+ 为什么要使用  Componsition API
+ 如何使用  Componsition API

1、什么是 Componsition API

```js
options api
export default {
  data() {
    return {
      key: value,
        功能A
        功能B
    };
  },
  methods: {
    name() {
          功能A
          功能B
    },
  },
  methods: {
    name() {
          功能A
          功能B
    },
  },
  watch: {
    data(newValue, oldValue) {},
  },
};
```





1、Componsiton Api  是根据逻辑相关性组织代码的，提高可读性和维护性

2、 更好地重用逻辑代码

vue2.0 是通过 Mixins 注入的形式来完成的=>容易发生命名冲突 且关系不清楚

+ 在vue3.0中 composition api是可选的

实践

逻辑复用

```js
// option-base
<template>
  <dialog0 :visible="showDialog0" @close="toggleDialog0"></dialog0>
  <dialog1 :visible="showDialog1" @close="toggleDialog1"></dialog1>
  <dialog2 :visible="showDialog2" @close="toggleDialog2"></dialog2>
  ...
  <dialog9 :visible="showDialog9" @close="toggleDialog9"></dialog9>
</template>

<script>
export default {
  data() {
    return {
      showDialog0: false,
      showDialog1: false,
      showDialog2: false,
      // ...
      showDialog9: false,
    }
  },
  
  methods: {
    toggleDialog0() {
      this.showDialog0 = !this.showDialog0
    },
    toggleDialog1() {
      this.showDialog1 = !this.showDialog1
    },
    toggleDialog0() {
      this.showDialog2 = !this.showDialog2
    },
    // ...
    toggleDialog0() {
      this.showDialog9 = !this.showDialog9
    },
  }
}
</script>
//
//这样，就能在一个页面上控制 10 这个类 dialog 的交互了。逻辑过于简单而忍不住 copy
//由于利用了 data 的响应式，我不能在外部抽象一个逻辑，去声明 show 个状态和返回一个改变状态的方法，这些逻辑算是在 option-base 的思维下最精简的代码了。
//当然对于这一系列状态可以用数组去优化，但是这些方法对于 composition-api 同样适用，所以不纳入比较范围，演示一下 composition-api 怎么复用这个响应式的状态

compostion api是如何做的
//useToggle.js
// reuse logic
export function useToggle(initState = false) {
  const state = ref(initState)
  const toggle = function(outerState) {
    if (typeof outerState === 'boolean') {
      state.value = outState
    } else {
      state.value = !state.value
    }
  }
  
  return {
    state,
    toggle
  }
}
//业务使用
// compostion-api
<template>
  <dialog0 :visible="showDialog0" @close="toggleDialog0"></dialog0>
  <dialog1 :visible="showDialog1" @close="toggleDialog1"></dialog1>
  <dialog2 :visible="showDialog2" @close="toggleDialog2"></dialog2>
  ...
  <dialog9 :visible="showDialog9" @close="toggleDialog9"></dialog9>
</template>

<script>
export default {
  setup() {
    const { state: showDialog0, toggle: toggleDialog0 } = useToggle()
    const { state: showDialog1, toggle: toggleDialog1 } = useToggle()
    const { state: showDialog2, toggle: toggleDialog2 } = useToggle()
    // ...
    const { state: showDialog9, toggle: toggleDialog9 } = useToggle()
    
    return {
      showDialog0,
      showDialog1,
      showDialog2,
      // ...
      showDialog9,
      toggleDialog0,
      toggleDialog1,
      toggleDialog2,
      // ...
      toggleDialog9,
    }
  },
}
</script>
//可以看到 composition-api 可以优雅的复用这些简单的逻辑，但是对于 vue-template 来说是没什么改变的。所以我更愿意把 composition-api 看作更加灵活利用 Vue 的方案，而不是 Vue 3.0 代码的语法变革。只不过刚好它随着 Vue 3.0 的实现而出现在大家眼前。
```



```javascript
import { ref, onMounted, onUnmounted } from 'vue'

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)

  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }

  onMounted(() => {
    window.addEventListener('mousemove', update)
  })

  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })

  return { x, y }
}

// 导入文件中
import { useMousePosition } from './mouse'

export default {
  setup() {
    const { x, y } = useMousePosition()
    // other logic...
    return { x, y }
  }
}





```

初始化时机

![](https://pic1.zhimg.com/80/v2-78a86c157451237f32878072b41c7907_720w.jpg)

生命周期的变化

```js
被替换

beforeCreate -> setup()
created -> setup()

重命名

beforeMount -> onBeforeMount
mounted -> onMounted
beforeUpdate -> onBeforeUpdate
updated -> onUpdated
beforeDestroy -> onBeforeUnmount
destroyed -> onUnmounted
errorCaptured -> onErrorCaptured

新增的
新增的以下2个方便调试 debug 的回调钩子：

onRenderTracked
onRenderTriggered

```

通信 vuex

```js
注入 
provide(TopGameDataSymbol, topGames);
provide(RecommendGameDataSymbol, recommendGames);
使用
const topGames = inject(TopGameDataSymbol);
const recommendGames = inject(RecommendGameDataSymbol);
```

与 React Hooks的对比

```java
基于函数的组合式 API 提供了与 React Hooks 同等级别的逻辑组合能力，但是与它还是有很大不同：组合式 API 的 setup() 函数只会被调用一次，这意味着使用 Vue 组合式 API 的代码会是：
一般来说更符合惯用的 JavaScript 代码的直觉；
不需要顾虑调用顺序，也可以用在条件语句中；
不会在每次渲染时重复执行，以降低垃圾回收的压力；
不存在内联处理函数导致子组件永远更新的问题，也不需要 useCallback；
不存在忘记记录依赖的问题，也不需要“useEffect”和“useMemo”并传入依赖数组以捕获过时的变量。Vue 的自动依赖跟踪可以确保侦听器和计算值总是准确无误。
```

