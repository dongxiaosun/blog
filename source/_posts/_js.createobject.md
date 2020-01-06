title: 对象的创建
meta:
  date: true
categories:
  - 教程
tags:
  - JS

---

虽然 Object 构造函数或者对象字面量都可以用来创建单个对象，但这些方式有个明显的缺点：使用相同的一个接口创建很多对象，会产生大量的重复代码。

<!-- more -->


## 工厂模式

```js
function createPerson(name, age, job) {
  var obj = {};
  obj.name = name;
  obj.age = age;
  obj.job = job;
  obj.sayName = function() {
    console.log(this.name);
  };
}

var sven = createPerson("sven", 23, "Software Engineer");
var greg = createPerson("greg", 25, "Software Engineer");
```
**优点：** 解决了创建多个相似对象的问题
**缺点：** 没有解决对象识的别问题(无法知道此对象的类型)


## 构造函数模式

```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function() {
    console.log(this.name);
  };
}

var sven = new Person("sven", 23, "Software Engineer");
var greg = new Person("greg", 25, "Software Engineer");
```

要创建 Person 的新实例，必须使用 new 操作符。以这种方式调用构造函数实际上会经历以下4个步骤：
> 1. 创建一个新对象
> 2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）
> 3. 执行构造函数中的代码（为这个新对象添加属性）
> 4. 返回新对象


#### constructor
constructor 指向创建当前对象的构造函数。

```js
console.log(sven.constructor === Person); // true
```

#### instanceof
instanceof 运算符用于测试构造函数的 prototype 属性是否出现在对象的原型链中的任何位置

```js
console.log(sven instanceof Person); // true
console.log(sven instanceof Object); // true
```

#### 构造函数的问题
使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍。在前面的例子中，sven 和 greg 都有一个名为 sayName 的方法，但那两个方法不是同一个 Function 的实例。ECMAScript 中的函数是对象，因此每定义一个函数，也就实例化了一个对象。从逻辑的角度讲，此时的构造函数也可以这样定义。
```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = new Function("console.log(this.name)"); // 与声明函数在逻辑上是等价的
}
```
从这个角度上来看构造函数，更容易明白每个 Person 实例都包含一个不同的 Function 实例的本质。说明白些，这种方式创建函数，会导致不同的作用域链和标识符解析，但创建 Function 新实例的机制仍是相同的。因此，同名实例上的同名函数是不相同的，以下代码可以证明这一点。

```js
console.log(sven.sayName == grey.sayName); // false
```
然而，创建两个完成同样任务的 Function 实例的确没有必要；况且有 this 对象在，根本不用在执行代码前就把函数绑定到特定对象上。因此，大家像下面这样，通过把函数定义转移到构造函数外部来解决这个问题。

```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
};

function sayName() {
  console.log(this.name);
}
```

在这个例子中，我们把 sayName 函数的定义转移到了构造 函数外部。在构造函数内部，我们将 sayName 属性设置成等价于全局的 sayName 函数。这样一来，由于 sayName 包含的是一个指向函数的指针，因此 sven 和 grey 对象就共享了在全局作用域中定义的一个 sayName 函数。这样的确解决了两个函数做一件事的问题，可新的问题又来了：在全局作用中定义的函数实际上只能被某个对象调用，这样全局作用域有点名不副实。

**优点：** 创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类
**缺点：** 使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍


## 原型模式

我们创建的**每个函数都有一个 prototype (原型)属性**，这个属性是一个指针，指向一个对象，而这个对象的用途是包含由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么 prototype 就是通过调用构造函数而创建的那个对象实例的原型对象。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面的例子所示。
```js
function Person () {};
Person.prototype.name = "sven";
Person.Prototype.age = 26;
Person.Prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
  console.log(this.name);
};

ver person1 = new Person();
person1.sayName(); // "sven"

var person2 = new Person()
person2.sayName(); // "sven"

console.log(person1.sayName == personName2); // true
```

在此，我们将 sayName() 方法和所有的属性直接添加到了 Person 的 prototype 属性中，构造函数变成了空函数。即使如此，也可以通过调用构造函数来创建新对象，而新对象还会具有相同的属性和方法。但与构造函数模式不相同的是，新对象的这些属性和方法是由所有实例共享的。换句话说，person1 和 person2 访问的都是同一组属性和同一个 sayName() 函数。要理解原型模式的工作原理，必须先理解 ECMAScript 中原型对象的性质。

#### 1. 理解原型对象
<i class="text-indent"></i>无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个 prototype 属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个 constructor (构造函数)属性，这个属性包含一个指向 prototype 属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor 指向 Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

<i class="text-indent"></i>创建了自定义的构造函数之后，其原型对象默认只会取得 constructor 属性;至于其他方法，则都是从 Object 继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针(内部属性)，指向构造函数的原型对象。ECMA-262第5版中，管这个指针叫 [[Prototype]]，虽然在脚本中没有标准的方式访问[[Prototype]],但 Firefox、Safari 和 Chrome 在每个对象上都支持一个属性 \_\_proto\_\_;而在其他实现中，这个属性对脚本则完全不可见的。不过，要明确的真正重要的一点就是，这个连接存在于实例与构造函数的原型对象之间，而不是存在于实例与构造函数之间。

<i class="text-indent"></i>以前面使用 Person 构造函数和 Person.prototype 创建实例的代码为例，下图展示了各个对象之间的关系。
![](/assets/createObj-1.jpg)
<font size="1"><center>各个对象之间的关系</center></font>

<i class="text-indent"></i>上图展示了 Person 构造函数、Person 的原型属性以及 Person 现有的两个实例之间的关系。在此，Person.prototype 指向了原型对象，而 Person.prototype.constructor 又指向了Person。原型对象中除了包含 constructor 属性之外，还包括后来添加的其他属性。Person的每个实例-- person1 和 person2 都包含一个内部属性，该属性仅仅指向了 Person.prototype; 换句话说，它们与构造函数没有直接的关系。此外，要格外注意的是，虽然这两个实例都不包含属性和方法，但我们缺可以调用 person1.sayName()。这是通过查找对象属性的过程来实现的。
<i class="text-indent"></i>虽然在所有现实中都无法访问到[[Prototype]],但是可以通过 isPrototypeOf() 方法来确定对象之间是否存在这种关系。


原型对象的方法 isPrototypeOf
返回对象的原型 Object.getPrototypeOf()
Object.getOwnPropertyDescriptor() 方法只能用于实例属性
hasOwnProperty() 可以检测一个属性是存在于实例中，还是存在于原型中。 在给定属性存在于对象实例中时，才会返回 true 。
原型与 in 操作符
在单独使用时， in 操作符会在通过对象能够访问给定属性时返回 true ，无论该属性存在于实例中还是原型中。

在使用 for-in 循环时，返回的是所有能够通过对象访问的、可枚举的（enumerated）属性，其中
既包括存在于实例中的属性，也包括存在于原型中的属性。

ECMAScript 5 的 Object.keys() 取得对象上所有可枚举的实例属性

要得到所有实例属性，无论它是否可枚举 使用 Object.getOwnPropertyNames()

## 组合使用构造函数模式和原型模式


## 动态原型模式

## 寄生构造函数模式

## 稳妥构造函数模式
