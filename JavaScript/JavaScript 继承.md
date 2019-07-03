**原型链继承**

利用原型让一个引用类型继承另一个引用类型的属性和方法。

缺点：引用类型的属性被所有实例共享，在创建子类型的实例时，没有办法在不影响所有对象实例的情况下给超类型的构造函数中传递参数。

**构造函数继承**

```javascript
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {
    Parent.call(this);
}	
```

缺点：方法都在构造函数中定义，每次创建实例都会创建一遍方法，没法函数复用。

优点：

- 避免了引用类型的属性被所有实例共享

- 可以在 Child 中向 Parent 传参

**组合继承**

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {

    Parent.call(this, name);

    this.age = age;

}

Child.prototype = new Parent();

var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

优点：融合原型链继承和构造函数的优点，是 JavaScript 中最常用的继承模式。

缺点：会调用两次父构造函数。一次是设置子类型实例的原型的时候，一次在创建子类型实例的时候。

**原型继承**

```javascript
function createObj(o) {
    function F(){}
    F.prototype = o;
    return new F();
}
```

模拟ES5 Object.create 的实现，将传入的对象作为创建的对象的原型

缺点： 包含引用类型的属性值始终都会共享相应的值，这点跟原型链继承一样。

**寄生式继承**

创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象。

```javascript
function createObj (o) {
    var clone = object.create(o);
    clone.sayName = function () {
        console.log('hi');
    }
    return clone;
}
```

缺点：跟借用构造函数模式一样，每次创建对象都会创建一遍方法；

**寄生组合式继承**

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法，基本思路：

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步
var F = function () {};

F.prototype = Parent.prototype;

Child.prototype = new F();


var child1 = new Child('kevin', '18');

console.log(child1);
```

最后我们封装一下这个继承方法：

```javascript
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

// 当我们使用的时候
prototype(Child, Parent)
```

引用《JavaScript高级程序设计》中对寄生组合式继承的夸赞就是：

这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

**ES6继承**

`Class` 可以通过extends关键字实现继承，如:

![img](https://user-gold-cdn.xitu.io/2019/7/1/16bae2f98aaba8f2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



对于ES6的 `class` 需要做以下几点说明：

1. 类的数据类型就是函数，类本身就指向构造函数。

```JavaScript
console.log(typeof SuperType);//function
console.log(SuperType === SuperType.prototype.constructor); //true
```

1. 类的内部所有定义的方法，都是不可枚举的。(ES5原型上的方法默认是可枚举的)

```JavaScript
Object.keys(SuperType.prototype);
```

1. `constructor` 方法是类的默认方法，通过 `new` 命令生成对象实例时，自动调用该方法。一个类必须有`constructor` 方法，如果没有显式定义，一个空的 `constructor` 方法会被默认添加。
2. `Class` 不能像构造函数那样直接调用，会抛出错误。

使用 `extends` 关键字实现继承，有一点需要特别说明：

- 子类必须在 `constructor` 中调用 `super` 方法，否则新建实例时会报错。如果没有子类没有定义 `constructor` 方法，那么这个方法会被默认添加。在子类的构造函数中，只有调用 `super` 之后，才能使用 `this`关键字，否则报错。这是因为子类实例的构建，基于父类实例，只有super方法才能调用父类实例。

学习自：

[刘小夕的文章](https://juejin.im/post/5d1a2814e51d4510835e02e9)

[冴羽的文章](https://juejin.im/post/591523588d6d8100585ba595)

