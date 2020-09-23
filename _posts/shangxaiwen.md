---
title: 执行上下文
date: 2020-02-23 16:39:56
tags: JavaScript
---

### 执行上下文

当执行 JS 代码时，会产生三种执行上下文

全局上下文

函数上下文

eval上下文

每个执行上下文中都有三个重要的属性

<!-- more -->

变量对象（`VO`），包含变量、函数声明和函数的形参，该属性只能在全局上下文中访问

作用域链（`JS` 采用词法作用域，也就是说变量的作用域是在定义时就决定了）

this

创建四部曲

创建AO对象

找形参和变量声明，将变量和形参作为AO属性名，值为undfined

将实参和形参统一

在函数体内找到函数声明，值赋给函数体

~~~js
 var a = 10
 function foo(i) {
   var b = 20
 }
 foo()
~~~

对于上述代码，执行栈中有两个上下文：全局上下文和函数 foo 上下文。

~~~js
stack = [
    globalContext,
    fooContext
]
~~~

对于全局上下文来说，`VO`大概是这样的

~~~js
globalContext.VO === globe
globalContext.VO = {
    a: undefined,
    foo: <Function>,
}
~~~

对于函数 `foo` 来说，`VO` 不能访问，只能访问到活动对象（`AO`）

~~~js
fooContext.VO === foo.AO
fooContext.AO {
    i: undefined,
    b: undefined,
    arguments: <>
}
// arguments 是函数独有的对象(箭头函数没有)
// 该对象是一个伪数组，有 `length` 属性且可以通过下标访问元素
// 该对象中的 `callee` 属性代表函数本身
// `caller` 属性代表函数的调用者
~~~

对于作用域链，可以把它理解成包含自身变量对象和上级变量对象的列表，通过 `[[Scope]]`属性查找上级变量

~~~js
fooContext.[[Scope]] = [
    globalContext.VO
]
fooContext.Scope = fooContext.[[Scope]] + fooContext.VO
fooContext.Scope = [
    fooContext.VO,
    globalContext.VO
]
~~~

接下来让我们看一个老生常谈的例子，`var`

~~~js
b() // call b
console.log(a) // undefined

var a = 'Hello world'

function b() {
    console.log('call b')
}
~~~

**以上的输出肯定都已经明白，这是因为函数和变量提升的原因。通常提升的解释是说将声明的代码移动到了顶部，这其实没有什么错误，便于大家理解。但是更准确的解释应该是：在生成执行上下文时，会有两个阶段。第一个阶段是创建的阶段（具体步骤是创建 `VO`），`JS` 解释器会找出需要提升的变量和函数，并且给他们提前在内存中开辟好空间，函数的话会将整个函数存入内存中，变量只声明并且赋值为 `undefined`，所以在第二个阶段，也就是代码执行阶段，我们可以直接提前使用。**

- 在提升的过程中，相同的函数会覆盖上一个函数，并且函数优先于变量提升

~~~js
b() // call b second

function b() {
    console.log('call b fist')
}
function b() {
    console.log('call b second')
}
var b = 'Hello world'
~~~

**`var`会产生很多错误，所以在 `ES6`中引入了 `let`。`let`不能在声明前使用，但是这并不是常说的 `let` 不会提升，`let` 提升了声明但没有赋值，因为临时死区导致了并不能在声明前使用。**

- 对于非匿名的立即执行函数需要注意以下一点

~~~js
var foo = 1
(function foo() {
    foo = 10
    console.log(foo)
}()) // -> ƒ foo() { foo = 10 ; console.log(foo) }
~~~

**因为当 `JS` 解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才可以访问到 `foo`，但是这个值又是只读的，所以对它的赋值并不生效，所以打印的结果还是这个函数，并且外部的值也没有发生更改。**

~~~js
specialObject = {};

Scope = specialObject + Scope;

foo = new FunctionExpression;
foo.[[Scope]] = Scope;
specialObject.foo = foo; // {DontDelete}, {ReadOnly}

delete Scope[0]; // remove specialObject from the front of scope chain
~~~

