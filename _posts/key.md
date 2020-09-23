---
title: Vue源码阅读回答一些常见问题
date: 2020-03-25 16:12:54
tags: vue
---

### 为什么data是一个函数

Vue组件可能存在多个实例，如果使用对象形式定义data，则会导致他们共用一个对象，那么状态变更的时候会影响所有的组件。这是不合理的，

采用函数形式定义，在initData时会将其作为全新的data对象，有效规避多实例之间的状态污染问题，而在vue跟实例过程中则不存在该限制，也是因为实例只有一个，不需要担心这个情况

<!-- more -->

### key的作用

源码部分src/core/vdom/patch.js

```
两个相同节点的比较过程
```

### v-for v-if的优先级

- v-for是优先与v-if被解析的可以查看vue源码部分 src/compiler/codegen/index

~~~js
if (el.staticRoot && !el.staticProcessed) {
        return genStatic(el, state)
    } else if (el.once && !el.onceProcessed) {
        return genOnce(el, state)
    } else if (el.for && !el.forProcessed) {
        return genFor(el, state)
    } else if (el.if && !el.ifProcessed) {
        return genIf(el, state)
    } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
        return genChildren(el, state) || 'void 0'
    } else if (el.tag === 'slot') {
        return genSlot(el, state)
    } else {
        // component or element
   可以看到解析了once后先解析for而后才是if
~~~

- 如果同时出现，每次渲染都会先执行循环再进行判断条件，无论如何循环都避免不了，浪费了性能
- 要避免出现这种情况，则在外层嵌套template，在这一层进行v-if的解析，然后内部进行循环

### 为什么组件必须是一个函数

源码部分 src/core/instance/state.js-initDate() 可以看到如果组件传过来的是一个对象，那么将会返回对象，而函数就会直接执行

~~~js
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
~~~

- Vue组件可能存在多个实例，如果使用对象形式定义data，则会导致他们共用一个对象，那么状态变更的时候会影响所有的组件。这是不合理的，

  采用函数形式定义，在initData时会将其作为全新的data对象，有效规避多实例之间的状态污染问题，而在vue跟实例过程中则不存在该限制，也是因为实例只有一个，不需要担心这个情况

### key的作用

源码部分src/core/vdom/patch.js

```js
  两个相同节点的比较过程
function sameVnode(a, b) {
    return (
        a.key === b.key && (
            (
                a.tag === b.tag &&
                a.isComment === b.isComment &&
                isDef(a.data) === isDef(b.data) &&
                sameInputType(a, b)
            ) || (
                isTrue(a.isAsyncPlaceholder) &&
                a.asyncFactory === b.asyncFactory &&
                isUndef(b.asyncFactory.error)
            )
        )
    )
}
更新过程
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
            if (isUndef(oldStartVnode)) {
                oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
            } else if (isUndef(oldEndVnode)) {
                oldEndVnode = oldCh[--oldEndIdx]
            } else if (sameVnode(oldStartVnode, newStartVnode)) {
                patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                oldStartVnode = oldCh[++oldStartIdx]
                newStartVnode = newCh[++newStartIdx]
            } else if (sameVnode(oldEndVnode, newEndVnode)) {
                patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
                oldEndVnode = oldCh[--oldEndIdx]
                newEndVnode = newCh[--newEndIdx]
            } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
                patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
                canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
                oldStartVnode = oldCh[++oldStartIdx]
                newEndVnode = newCh[--newEndIdx]
            } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
                patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
                oldEndVnode = oldCh[--oldEndIdx]
                newStartVnode = newCh[++newStartIdx]
            } else {
                if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
                idxInOld = isDef(newStartVnode.key) ?
                    oldKeyToIdx[newStartVnode.key] :
                    findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
                if (isUndef(idxInOld)) { // New element
                    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
                } else {
                    vnodeToMove = oldCh[idxInOld]
                    if (sameVnode(vnodeToMove, newStartVnode)) {
                        patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
                        oldCh[idxInOld] = undefined
                        canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
                    } else {
                        // same key but different element. treat as new element
                        createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
                    }
                }
                newStartVnode = newCh[++newStartIdx]
            }
        }
        if (oldStartIdx > oldEndIdx) {
            refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
            addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
        } else if (newStartIdx > newEndIdx) {
            removeVnodes(oldCh, oldStartIdx, oldEndIdx)
        }
    }

```

- 可以看到key的作用
  - 主要是为了高效更新虚拟dom，其原理是vue在path的过程中可以精确的判断两个节点是否是同一个，从而避免频繁的更新不同元素，使得整个path过程变得更加高效，减少dom操作量，提高性能
  - 另外若不设置key，还可能在列表更新时引发一些隐藏的bug
  - vue中使用相同签名元素的过渡切换时，也会使用到key，其目的也是为了让vue可以区分它们。否则vue只会替换，其内部属性而不会被触发过渡效果
  - 例如更新一个列表 在中间插入一个新的节点 不加key的过程是 当标签名称相同就认为是相同节点然后更新当找到要插入的节点时会直接覆盖旧的节点 然后后面的节点都变的不同了也都会更新 而加了key之后会比对新旧节点 key会记住自身的状态 当这个数组处理结束后发现只有新插入的节点没有匹配 并且在之前遍历的时候会记住他应该插入的位置，此时会创建新的节点然后插入到所记住的位置

### diff算法

- diff算法是虚拟dom的必然产物，通过新旧dom的对比(即diff)，将变化的地方更新在真实的dom上，另外，也需要diff高效的执行对比过程，从而降低时间复杂度为O(n)
- vue2.x中为了降低Wacher粒度，每个组件只有一个Wacher与之对应，只有引入diff才能精确找到发生变化的地方。
- vue中diff执行的时刻是组件实例执行更新函数时，他会对上一次渲染的结果oldVdom和新的渲染结果newVdom，此过程称为patch
- diff过程中遵循深度优先，同层策略的策略，两个节点之间会根据他们是否拥有子节点或者文本节点做不同的操作，比较两组子节点是算法的重点，首先假设头尾节点可能相同做4次比对尝试，如果没有找到相同节点才按照通用的方式遍历查找，查找结果再按情况处理剩下的节点，借助key通常可以非常精确的找到相同节点，因此整个path过程非常高效

### 组件

- 组件是独立和可复用的组织单元，组件系统是Vue核心特征之一，他使开发者使用小型，独立和通常可复用的组件构建大型应用
- 组件化开发能大幅度提升开发效率，测试性，复用性等；
- 组件使用按照分类有：页面组件，业务组件，通用组件
- vue组件是基于配置的我们通常编写的组件其实是组件的配置而非组件，框架后续会生成构造函数，他们基于VueComponent扩展于Vue
- Vue中常见的组件化技术有： 属性props、自定义事件、插槽等他们主要用于组件通信，扩展等
- 合理的划分组件。有助于提升应用性能
- 组件应该是高内聚，低耦合的
- 组件遵循单向数据流

~~~js
Vue.extend = function(extendOptions: Object): Function {
        extendOptions = extendOptions || {}
        const Super = this
        const SuperId = Super.cid
        const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
        if (cachedCtors[SuperId]) {
            return cachedCtors[SuperId]
        }

        const name = extendOptions.name || Super.options.name
        if (process.env.NODE_ENV !== 'production' && name) {
            validateComponentName(name)
        }
        //创建一盒Vuecomponent的类
        const Sub = function VueComponent(options) {
                this._init(options)
            }
            // 继承vue
        Sub.prototype = Object.create(Super.prototype)
        Sub.prototype.constructor = Sub
        Sub.cid = cid++
            // 选项合并
            Sub.options = mergeOptions(
                Super.options,
                extendOptions
            )
        Sub['super'] = Super
~~~

