---
title: 分享一些有趣的 Web APIs
date: 2021-05-01 14:52:30
tags: JavaScript
---

## Web API 简介

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API

在使用 JavaScript 编写 Web 应用时，有许多 Web API 可供调用。

## 一些有趣的 Web APIs

<!-- more -->

### Element.scrollIntoView

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollIntoView

定义：`Element.scrollIntoView()` 方法让当前的元素滚动到浏览器窗口的可视区域内。

场景：锚点跳转、回到顶部、去到底部

使用：

```javascript
element.scrollIntoView() // 等同于 element.scrollIntoView(true)
element.scrollIntoView(alignToTop) // Boolean 型参数
element.scrollIntoView(scrollIntoViewOptions) // Object型参数

// 参数 alignToTop 是一个 Boolean 值，用来指定对齐方式
// 参数 scrollIntoViewOptions 是一个对象，用来定义动画过渡效果和对齐方式
```

补充：该 API 还有一个变体，`Element.scrollIntoViewIfNeeded()` 方法用来将不在浏览器窗口的可见区域内的元素滚动到浏览器窗口的可见区域。

示例：回到顶部

```javascript
const rootEl = document.getElementsByTagName('html')[0]

if (rootEl.scrollIntoView) {
  rootEl.scrollIntoView({ behavior: 'smooth' })
  return
}

// Fix incompatible browser
document.documentElement.scrollTop = document.body.scrollTop = 0
```

示例：去到底部

```javascript
const rootEl = document.getElementsByTagName('html')[0]

if (rootEl.scrollIntoView) {
  rootEl.scrollIntoView({ behavior: 'smooth', block: 'end' })
  return
}

// Fix incompatible browser
const documentHeight = document.body.offsetHeight || document.body.scrollHeight
document.documentElement.scrollTop = document.body.scrollTop = documentHeight
```

示例：滚动到指定元素：

```javascript
const targetBtn = document.getElementById('target')
const targetEl = document.getElementById('targetEl')

targetBtn.addEventListener('click', () => {
  targetEl.scrollIntoViewIfNeeded({ behavior: 'smooth' })
})
```

### Page Visibility API

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API/Page_Visibility_API

定义：Page Visibility API 用来查看当前页面的可见性状态。

场景：我们在很多地方都需要判断用户是不是在当前页面，如果离开了当前页面我们需要捕捉到并进行一些操作。例如：当视频处于播放状态时，我们需要判断用户是不是在当前页面以继续播放，如果离开了我们需要暂停播放。

使用：当用户最小化窗口或切换到另一个选项卡时，浏览器会触发一个 `visibilitychange` 事件，让监听者知道页面状态已更改，你可以监听该事件并执行某些操作。使用 `document.hidden、document.visibilityState` 属性查看页面可见性状态。

```javascript
// 监听页面可见属性的改变
// 如果页面是隐藏状态，则暂停视频；如果页面是展示状态，则播放视频
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    videoEl.pause()
  } else {
    videoEl.play()
  }
})
```

兼容性代码：

```javascript
let hidden, visibilityChange
if (typeof document.hidden !== 'undefined') {
  hidden = 'hidden'
  visibilityChange = 'visibilitychange'
} else if (typeof document.msHidden !== 'undefined') {
  hidden = 'msHidden'
  visibilityChange = 'msvisibilitychange'
} else if (typeof document.webkitHidden !== 'undefined') {
  hidden = 'webkitHidden'
  visibilityChange = 'webkitvisibilitychange'
}
```

### Intersection Observer API

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API

定义：检测元素的可视状态或者两个元素的相对可视状态。

这个 API 覆盖最广的场景是“如果两个元素发生的交集部分在 N% 左右，我需要做处理一些事情（执行回调）”

场景：

- 当页面滚动时，懒加载图片或请求接口。
- 商品列表页，监听页面可视区域内的商品，然后记录商品曝光次数；
- 为计算广告收益，检测其广告元素的曝光情况；
- 根据用户是否能看见相应区域来开始执行任务或动画。

使用：Intersection Observer API 允许你配置一个回调函数，每当目标元素（target）与设备视窗（root）或者其他指定元素发生交集的时候执行。

```javascript
const options = {
  root: document.querySelector('#scrollArea'),
  rootMargin: '0px',
  // 阈值为 0   意味着目标元素 **开始出现** 在 root 选项指定的元素中时，回调函数将会被执行。
  // 阈值为 1.0 意味着目标元素 **完全出现** 在 root 选项指定的元素中时，回调函数将会被执行。
  threshold: 0,
}

const callback = (entries, observer) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      console.log('元素可见')
    } else {
      console.log('元素不可见')
    }
  })
}

// 创建一个 IntersectionObserver对象，并传入相应参数和回调用函数，
// 该回调函数将会在目标(target)元素和根(root)元素的交集大小超过阈值(threshold)规定的大小时候被执行。
const observer = new IntersectionObserver(callback, options)

let target = document.querySelector('#listItem')
observer.observe(target)
```

### FullScreen API

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API

定义：FullScreen API 可以让一个元素与其子元素占据整个屏幕，并在此期间，从屏幕上隐藏所有的浏览器用户界面以及其他应用。

场景：幻灯片演讲、网页游戏、信息流网站、视频等

使用：

1. 浏览器是否支持全屏模式：`document.fullscreenEnabled`
2. 进入全屏模式：`Element.requestFullscreen()`
3. 退出全屏模式：`document.exitFullscreen()`
4. 检查当前是否有节点处于全屏状态：`document.fullscreenElement`
5. 进入/离开全屏时 触发事件：`fullscreenchange`
6. 进入全屏失败时 触发事件: `fullscreenerror`

Chrome 下的全屏表现：

1. 隐藏标签栏、书签栏
2. 将要全屏的元素充满整个屏幕，并因此其他元素
3. 可以多层全屏，当出现多层全屏的情况，需要一层层的退出到页面最初始的情况
4. 进入全屏时，浏览器有一个默认的提示：按 ESC/F11 即可退出全屏模式

```javascript
// 这是 FullScreen API 的演示，在此例中，网页中显示了一个视频。
// 按下 Enter 键让用户在视频的窗口显示和全屏显示之间切换。
function toggleFullScreen() {
  // 检查当前是否有节点处于全屏状态
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen()
  } else {
    if (document.exitFullscreen) {
      document.exitFullscreen()
    }
  }
}

document.addEventListener('keydown', (e) => {
  if (e.keyCode == 13) {
    toggleFullScreen()
  }
})
```

兼容性代码：

```JAV
function toggleFullScreen() {
  if (!document.mozFullScreen && !document.webkitFullScreen) {
    if (videoEl.mozRequestFullScreen) {
      videoEl.mozRequestFullScreen()
    } else {
      videoEl.webkitRequestFullScreen(Element.ALLOW_KEYBOARD_INPUT)
    }
  } else {
    if (document.mozCancelFullScreen) {
      document.mozCancelFullScreen()
    } else {
      document.webkitCancelFullScreen()
    }
  }
}
```

多层全屏：

```javascript
const outerBtn = document.getElementById('outer')
const innerBtn = document.getElementById('inner')
const outerBox = document.querySelector('.outer-box')
const innerBox = document.querySelector('.inner-box')

outerBtn.addEventListener('click', () => {
  outerBox.requestFullscreen()
})
innerBtn.addEventListener('click', () => {
  innerBox.requestFullscreen()
})
```

### Notification API

> MDN 文档：https://developer.mozilla.org/zh-CN/docs/Web/API/notification

![img](https://user-gold-cdn.xitu.io/2020/7/8/1732d7af8e76e22e?w=397&h=103&f=png&s=5142)

定义：Notification API 是 HTML5 新增的桌面通知 API，用于向用户显示通知信息。该通知是脱离浏览器的，即使用户没有停留在当前标签页，甚至最小化了浏览器，该通知信息也一样会置顶显示出来。

场景：发送用户订阅的消息，网站内容更新提醒等，提醒有人关注了。

使用：实例化一个 `Notification` 对象即可。

假定有如下 HTML：

```javascript
<button onclick="notifyMe()">Notify me!</button>
```

接下来发送一条通知：

```javascript
function notifyMe() {
  // 先检查浏览器是否支持
  if (!("Notification" in window)) {
    alert("This browser does not support desktop notification");
  }

  // 检查用户是否同意接受通知
  else if (Notification.permission === "granted") {
    // If it's okay let's create a notification
    var notification = new Notification("Hi there!");
  }

  // 否则我们需要向用户获取权限
  else if (Notification.permission !== 'denied') {
    Notification.requestPermission(function (permission) {
      // 如果用户同意，就可以向他们发送通知
      if (permission === "granted") {
        var notification = new Notification("Hi there!");
      }
    });
  }


  // 最后，如果执行到这里，说明用户已经拒绝对相关通知进行授权
  // 出于尊重，我们不应该再打扰他们了
}
```