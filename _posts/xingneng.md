---
title: 性能优化的一个小case
date: 2020-03-20 16:38:49
tags: 性能优化
---

### 先说两个杀手级概念

`reflow` 回流，或者叫重排都可以。回流(reflow)这个名词指的是浏览器为了重新渲染部分或全部的文档而重新计算文档中元素的位置和几何结构的过程。

简单来说就是当页面布局或者几何属性改变时就需要reflow。

在一个页面中至少在页面刚加载的时候有一次reflow，在reflow的过程中浏览器会将render tree中受影响的节点失效，再重新构建render tree，有时候，即使仅仅回流一个单一的元素，也可能要求它的父元素以及任何跟随它的元素也产生回流

<!-- more -->

`repaint`重绘，当页面中的元素只需要更新样式风格不影响布局，比如更换背景色background-color，这个过程就是重绘。

如何触发应该都知道吧！

问题来了我看到用transtion代替top感到一阵懵逼，这是为什么呢？？？代码说话！

~~~html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #react {
            position: relative;
            top: 0;
            width: 100px;
            height: 100px;
            background-color: red;
        }
    </style>
</head>

<body>
    <div id="react"></div>
    <script type="text/javascript">
        setTimeout(() => {
            document.getElementById("react").style.top = "100px"
        }, 2000);
    </script>
</body>
</html>

~~~

代码很简单，页面上有一个红色的方块，2秒后它的top值将会变为“100px”，为了方便体现替代的属性可以避免reflow这里我们使用chrome的开发者工具，部分截图如下

![](https://i.loli.net/2020/05/07/6aZST9eD7McLHoq.jpg)

如上图，在top值变为“100px”的过程中有上图五个阶段.

- Recalculate Style，浏览器计算改变过后的样式
- Layout，这个过程就是我们说得reflow回流过程
- Update Layer Tree，更新Layer Tree
- Paint，图层的绘制过程
- Composite Layers,合并多个图层

我们把这五个过程用时记下：80 + Layout(73) + 72 + 20 + 69 = **316us**

再用translate替代top：

~~~js
-       position: relative;
-       top: 0;
+       transform: translateY(0);

-       document.getElementById("react").style.top = "100px"
+       document.getElementById("react").style.transform = "translateY(100px)"

~~~

![](https://i.loli.net/2020/05/07/XWp9CAo3gnbVYy7.jpg)

可以看到用translate替换top后减少了原来的Layout也就是reflow的过程，用时：81 + 80 + 36 + 83 = **280us**。 结果非常明显315us减少到了280us。有人说这个效果不明显呀，但是让我们设想这样一个业务场景，有许多网站都会有不停移动的飘窗，这种飘窗通常是用定时器实现，每隔100ms就去修改一次它的top，如果用translate的话1s就可以减少10次reflow，如果这个飘窗样式比较多，比较复杂，那么1秒钟减少的10次reflow就有可能**减少几百毫秒甚至几秒Layout的过程**。

##### transform

transform 属于合成属性（composite property），对合成属性进行 transition/animation 动画将会创建一个合成层（composite layer），这使得被动画元素在一个独立的层中进行动画。通常情况下，浏览器会将一个层的内容先绘制进一个位图中，然后再作为纹理（texture）上传到 GPU，只要该层的内容不发生改变，就没必要进行重绘（repaint），浏览器会通过重新复合（recomposite）来形成一个新的帧。

##### margin top / left

top/left属于布局属性，该属性的变化会导致重排（reflow/relayout），所谓重排即指对这些节点以及受这些节点影响的其它节点，进行CSS计算->布局->重绘过程，浏览器需要为整个层进行重绘并重新上传到 GPU，造成了极大的性能开销。

总算是明白了！