---
title: js常见继承的实现方式
date: 2020-04-26 16:15:25
tags: JavaScript
---

#### 原型链继承

~~~js
// 借用原型链实现继承
function Animal() {
    this.name = 'alex';

}
Animal.prototype.getName = function() {
    return this.name
}

function Dog() {}
// 本质 重写原型对象
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog;
var d1 = new Dog();
~~~

- 特点： 重写子类的原型对象，父类原型对象上的属性和方法都会被子类继承
- 缺点： 在父类中定义的实例引用类型的属性，一旦被修改，其他的实例也会被修改。当实例化子类的时候，不能传递参数到父类

<!-- more -->

#### 构造函数式继承

~~~js
// 借用构造函数实现
function Animal(name) {
    this.name = name;
    this.color = ['red', 'blue']

}
Animal.prototype.getName = function() {
    return this.name
}

function Dog(name) {
    //继承了父类的共享属性
    Animal.call(this, name)
}
var d1 = new Dog('tom');
~~~

- 特点： 在子类构造函数内部空间调用（call aply bind）父类的构造函数
- 原理：改变了this的指向
- 优点： 仅仅把父类中的实例属性当做子类的实例属性，并且还能传参数
- 父类中公有的方法不能被继承

#### 组合式继承

~~~js
/ 组合继承 = 原型链+构造函数
function Animal(name) {
    this.name = name;
    this.color = ['red', 'blue']

}
Animal.prototype.getName = function() {
    return this.name
}

function Dog(name) {
    Animal.call(this, name)
}
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog;
var d1 = new Dog('tom');
var d2 = new Dog('sexx');
~~~

- 特点： 结合了原型链继承和借阅构造函数继承的优点原型链继承：公有的方法能被继承下来借用构造函数继承：实例属性能被继承下来
- 缺点：调用了两次父类的构造函数
  - 1 实例化子类对象
  - 2 子类的构造函数内部（好）

### 寄生组合式继承

~~~js
// 寄生组合式继承
function Animal(name) {
    this.name = name;
    this.color = ['red', 'blue']

}
Animal.prototype.getName = function() {
    return this.name
}

function Dog(name) {
    Animal.call(this, name)
}
// 重写原型对象：把父类的共享方法继承下来
Dog.prototype = Object.create(Animal.prototype)
Dog.prototype.constructor = Dog;
var d1 = new Dog('tom');
var d2 = new Dog('sexx');
~~~

- **var a =Object.create(a) 将a对象作为b实例的原型对象 把子类的原型对象指向了父类的原型对象Dog.prototype = Object.create(Animal.prototype)**

这是实际开发中使用最多的一种模式