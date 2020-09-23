---
title: class
date: 2020-03-22 21:54:08
tags: Es6
---

  `super`这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

1:第一种情况：super`作为函数调用时，代表父类的构造函数。ES6 要求，子类的构造函数必须执行一次`super`函数。注意,作为函数时，super() 只能用在子类的构造函数之中，用在其他地方就会报错。

<!-- more -->

~~~js
class A {}
class B extends A {
  constructor() {
    super();
  }
}
~~~

super 作为函数调用时，内部的 this 指的是子类实例,注意，`super`虽然代表了父类`A`的构造函数，但是返回的是子类`B`的实例，即`super`内部的`this`指的是`B`的实例，因此`super()`在这里相当于`A.prototype.constructor.call(this)`。

~~~js
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
~~~

上面代码中，`new.target`指向当前正在执行的函数。可以看到，在`super()`执行时，它指向的是子类`B`的构造函数，而不是父类`A`的构造函数。也就是说，`super()`内部的`this`指向的是`B`。

作为函数时，`super()`只能用在子类的构造函数之中，用在其他地方就会报错。

~~~js
class A {}

class B extends A {
  m() {
    super(); // 报错
  }
}
~~~



上面代码中，`super()`用在`B`类的`m`方法之中，就会造成语法错误。

2:第二种情况，`super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```js
 class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

上面代码中，子类`B`当中的`super.p()`，就是将`super`当作一个对象使用。这时，`super`在普通方法之中，指向`A.prototype`，所以`super.p()`就相当于`A.prototype.p()`。

这里需要注意，由于`super`指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过`super`调用的。

```js
  class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

上面代码中，`p`是父类`A`实例的属性，`super.p`就引用不到它。

如果属性定义在父类的原型对象上，`super`就可以取到。

```js
  class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

上面代码中，属性`x`是定义在`A.prototype`上面的，所以`super.x`可以取到它的值。

ES6 规定，在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类实例。

需要注意的地方

1： 由于`this`指向子类实例，所以如果通过`super`对某个属性赋值，这时`super`就是`this`，赋值的属性会变成子类实例的属性。

```js
  class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

上面代码中，`super.x`赋值为`3`，这时等同于对`this.x`赋值为`3`。而当读取`super.x`的时候，读的是`A.prototype.x`，所以返回`undefined`。

2：如果`super`作为对象，用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。

```js
 class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

总结一下：super有两种使用方法

（1）作为函数

- super指向父类的构造函数，super内部的this指向子类的实例

（2）作为对象

- super指向父类的原型对象，而不是构造函数，
- 这时候又分为两种情况 在普通方法中，指向父类的原型对象；在静态方法中，指向父类本身。