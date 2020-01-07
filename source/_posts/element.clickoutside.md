title: Element源码学习--指令 v-clickoutside
date: 2019-04-18
categories:
  - 源码
tags:
  - Vue
  - Element

---

Element源码学习--指令 v-clickoutside

<!-- more -->

clickoutside 是 Element-ui 实现的一个自定义指令，顾名思义，该指令用来处理目标节点之外的点击事件，常用来处理下拉菜单等展开内容的关闭，在Element-ui 的 Select 选择器、Dropdown 下拉菜单、Popover 弹出框等组件中都用到了该指令，所以这个指令在实现一些自定义组件的时候非常有用。

## Vue 自定义指令

要分析该源码，首先要了解一下 Vue 的自定义指令。自定义指令的定义方式如下：
```js
// 注册一个全局自定义指令 
Vue.directive("directiveName", {
  bind: function(el, binding, vnode){
  // 当指令第一次绑定到元素时调用，常用来进行一些初始化设置
  // 代码
  },
  inserted: function(el, binding, vnode){
  // 当被绑定的元素插入到 DOM 中时……
  // 代码
  },
  update: function(el, binding, vnode, oldVnode){
  // 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前
  // 代码
  },
  componentUpdated: function(el, binding, vnode, oldVnode){
  // 指令所在组件的 VNode 及其子 VNode 全部更新后调用
  // 代码
  },
  unbind: function(el, binding, vnode){
  // 只调用一次，指令与元素解绑时调用，类似于beforeDestroy的功能
  // 代码
  }
});
```


可以看到在配置对象中只有5个可选的钩子函数，他们的参数有4个，分别是： **el、binding、vnode、oldVnode**

**el:** 指令所绑定的元素，可以用来直接操作 DOM
**binding:** 一个包含了自定义详细信息的对象，内部收集了使用自定义指令时传入的值、修饰符、参数等数据，详细信息可以在官方文档见到，已经说的十分详细了
**vnode:** Vue编译生成的虚拟节点
**oldVnode:** 本次Vnode更新之前，上一次产生的虚拟节点，仅在  update  和  componentUpdated  钩子中可用。

#### 看完了自定义指令的内容，接下来我们就来分析 clickoutside 的具体实现。


```js
import Vue from "vue";
import { on } from "element-ui/src/utils/dom";

const nodeList = [];
const ctx = "@@clickoutsideContext";

let startClick;
let seed = 0;

!Vue.prototype.$isServer && on(document, "mousedown", e => (startClick = e));

!Vue.prototype.$isServer && on(document, "mouseup", e => {
  nodeList.forEach(node => node[ctx].documentHandler(e, startClick));
});

function createDocumentHandler(el, binding, vnode) {
  return function(mouseup = {}, mousedown = {}) {
  // 代码
  };
}

export default {
  bind(el, binding, vnode) {
  // 代码
  },

  update(el, binding, vnode) {
  // 代码
  },

  unbind(el) {
  // 代码
  }
};
```

上面是简化后的源码，可以看到首先引入 Vue 和一个用来进行事件绑定的工具函数 on，然后定义了两个全局常量 nodeList 和 ctx 。nodeList 是一个元素搜集器 ，会将页面中所有绑定了 clickoutside 指令的dom元素存储起来，而ctx定义了一个命名空间（必须比较特殊，防止和其它特性重名）， 后面会将它添加为元素 el 的 properties ，具体后面会分析到。

接着利用之前引入的Vue进行判断，非服务端则给文档对象添加 mousedown 和 mouseup 事件，在 mousedown 事件回调中，将事件对象存储到 startClick 全局变量中，在 mouseup 事件回调中遍历 nodeList ，然后分别执行每一个 node ( 即之前存储起来的 clickoutside 指令绑定的元素 el ) ctx 特性中存储的 documentHandler 函数 。关于 ctx property 的值会在后面介绍。


最后就是导出了一个 clickoutside 的配置对象，在用到 clickoutside 指令的组件中导入该配置对象，然后在组件中局部注册后就可以使用了。

该配置对象中使用了 bind、update、unbind 三个钩子函数来定义 clickoutside 指令，主要做的事情就是搜集该自定义指令的相关信息，然后存储到 el 的 ctx 特性上。接下来具体来看一下这个搜集过程。

首先是bind钩子函数：

```js
bind(el, binding, vnode) {
  nodeList.push(el);
  const id = seed++;
  el[ctx] = {
    id,
    documentHandler: createDocumentHandler(el, binding, vnode),
    methodName: binding.expression,
    bindingFn: binding.value
  };
}

```
这里首先将 el 直接 push 到 nodeList 中，这样每次有 clickoutside 指令绑定到页面上，都会将绑定元素存储到 nodeList 当中去，即前面说过的 元素搜集器 。接下来将全局变量 seed++，并且赋值给一个临时变量 id，最后就是给 el 的 ctx 特性赋值了，它的值是一个对象，内部包括了:

id：前面生成的全局唯一 id ，用来标识该 clickoutside 指令

documentHandler：利用 createDocumentHandler 生成的一个回调函数。前面的分析中说到，给页面绑定的 mouseup 事件回调中，会遍历 nodeList，分别执行每一个绑定元素 el 的 ctx 特性上的 documentHandler 函数， 这个函数就是在这里生成的 ，至于这个回调函数究竟是做了什么，后面再详细分析。

methodName：binding.expression，查看自定义指令的文档可以知道， binding.expression 的值是字符串形式的指令表达式。例如有`<div v-my-directive="1 + 1"></div> `，则 binding.expression 的值为 1 + 1


bindingFn：binding.value，指令的绑定值，还是上面的例子，则 binding.value 的值是 2 （1 + 1等于2），即：指令的值为 js 表达式的情况下， **binding.expresssion** 为表达式本身，是一个字符串，而 **binding.value** 是该表达式的值。

接着我们看下 update 钩子：

```js
update(el, binding, vnode) {
	el[ctx].documentHandler = createDocumentHandler(el, binding, vnode);
	el[ctx].methodName = binding.expression;
	el[ctx].bindingFn = binding.value;
}
```


可以看到 update 钩子的内容很简单，就是当组件更新的时候，更新绑定元素 el 的特性 ctx 中的值。

再接着我们看看最后一个钩子 unbind :

```js
unbind(el) {
  let len = nodeList.length;

  for (let i = 0; i < len; i++) {
    if (nodeList[i][ctx].id === el[ctx].id) {
      nodeList.splice(i, 1);
      break;
    }
  }
  delete el[ctx];
}
```


这个钩子也很简单，就是当 clickoutside 指令与元素el解绑的时候，遍历 nodeList ，通过 ctx 特性上的 id 找到 nodeList 中存储的当前解绑元素 el，将它从 nodeList 中删除，并且删除 e l上的 ctx 特性。

以上就是 clickoutside 指令配置对象中做的所有操作，总结起来就是：

当指令与元素绑定以及组件更新的时候，搜集并设置绑定元素的 ctx 特性，同时将绑定元素添加到 nodeList 当中去，当指令与元素解绑的时候，删除 nodeList 中存储的对应的绑定元素，并将之前设置在绑定元素上之前设置的ctx特性删除掉。

前面说过，给页面绑定的 mouseup 事件回调中，会遍历 nodeList，分别执行搜集起来的每一个绑定元素 el 上的 ctx 特性中的 documentHandler 函数。而该函数是通过  createDocumentHandler 函数生成的，让我们看看这个函数都做了什么。

```js
function createDocumentHandler(el, binding, vnode) {
  return function(mouseup = {}, mousedown = {}) {
    if (!vnode ||
      !vnode.context ||
      !mouseup.target ||
      !mousedown.target ||
      el.contains(mouseup.target) ||
      el.contains(mousedown.target) ||
      el === mouseup.target ||
      (vnode.context.popperElm &&
      (vnode.context.popperElm.contains(mouseup.target) ||
      vnode.context.popperElm.contains(mousedown.target)))) return;

    if (binding.expression &&
      el[ctx].methodName &&
      vnode.context[el[ctx].methodName]) {
      vnode.context[el[ctx].methodName]();
    } else {
      el[ctx].bindingFn && el[ctx].bindingFn();
    }
  };
}
```
**contains**： 方法就是比较实用的函数，如果 A 元素包含B元素，则返回true，否则 false ，这对于判断元素的位置很重要。


可以看到，这个函数利用了闭包将传入的参数缓存起来，然后返回一个函数。在这个返回的函数中，会进行一系列判断，首先在第一个if里面，判断了：

vnode.context 是否存在，不存在退出
mouseup.target 是否存在，不存在退出
mousedown.target 是否存在，不存在退出
绑定对象el是否包含 mouseup.target/mousedown.target 子节点，如果包含说明点击的是绑定元素的内部，则不执行 clickoutside 指令内容
绑定对象el是否等于 mouseup.target ，等于说明点击的就是绑定元素自身，也不执行 clickoutside 指令内容
最后 vnode.context.popperElm 这部分内容则是: 判断是否点击在下拉菜单的上，如果是，也是没有点击在绑定元素外部，不执行 clickoutside 指令内容


如图，如果点击在红色区域内，则全部不触发 clickoutside 指令的逻辑。

![](/assets/image/element-clickoutside.jpg)

如果以上条件全部符合，则判断闭包缓存起来的值，如果 methodName 存在则执行这个方法，如果不存在则执行 bindingFn 。例如：


```html
<template>
	<div v-clickoutside="handleClose"></div>
</template>

<script>
export default {
  data(){
    return {
    visible: false
    };
  },

  methods: {
    handleClose(){
    this.visible = false;
    }
  }
}
</script>
```


在这个例子中， methodName 或者 bindingFn 就是通过指令传入的 handleClose 方法。执行该方法，就可以执行 clickoutside 指令的逻辑了

以上就是 documentHandler 方法的生成以及内部逻辑。通过这个方法和之前的分析，我们就可以知道，当页面绑 mouseup 事件触发的时候，会遍历 nodeList ，依次执行每一个绑定元素 el 的 ctx 特性上的 documentHandler 方法。而在这个方法内部可以访问到指令传入的表达式，在进行一系列判断之后会执行该表达式，从而达到点击目标元素外部执行给定逻辑的目的，而这个给定逻辑是通过自定义指令的值，传到绑定元素 el 的 ctx 特性上的。

至此 clickoutside 的源码就分析完了，可以看到 clickoutside 指令的源码并不复杂，不过涉及到的内容还是挺多的，有许多东西值得我们学习，比如利用 dom 元素的特性来存储额外信息，使用闭包缓存变量，如何判断点击在目标元素外部和 Vue 自定义指令的使用等等。