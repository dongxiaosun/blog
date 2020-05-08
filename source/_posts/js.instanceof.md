title: instanceof 详解 # 文章标题
date: 2020-05-08 # 创建时间
abbrlink: js.instanceof # 文章名称
categories:
  - 学习 # 分类
tags:
  - JS # 标题

---

instanceof 详解

<!-- more -->

### MDN 解释

`instanceof` 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。

### 语法

```
object instanceof constructor
```

例如：

```js
console.log(Object instanceof Function); // true
console.log(Function instanceof Object); // true
console.log(Function instanceof Function); //true
```

`instanceof` 的判断规则是沿着第一参数的`__proto__`这条线来找，同时沿着第二个参数的`prototype`这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回 true，如果找到终点还未重合，则返回 false。

![](/assets/image/js.instanceof.png)

**参考资料：**

[本节的图片来源：http://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/figure1.jpg](http://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/figure1.jpg)
