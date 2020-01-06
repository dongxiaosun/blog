title: vue 中的权限设计
meta:
  date: true
categories:
  - 教程
tags:
  - Vue

---

学习 vue-router 完成的导航解析流程

<!-- more -->

## 路由导航分类

+ 1.全局的。
+ 2.单个路由独享。
+ 3.组件级的。

## vue 中导航的流转如下所示：

+ 1.导航被触发。
+ 2.在失活的组件里调用离开守卫。
+ 3.调用全局的 `beforeEach` 守卫。
+ 4.在`重用的组件`里调用 `beforeRouteUpdate` 守卫（2.2+）。
+ 5.在路由配置里调用 `beforeEnter`。
+ 6.解析异步路由组件。
+ 7.在被激活的组件里面调用 `beforeRouterEnter`。
+ 8.调用全局的 `beforeResolve` 守卫（2.5+）。
+ 9.导航被确认。
+ 10.调用全局的 `afterEach`钩子。
+ 11.触发 `DOM` 更新。
+ 12.用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。


## 如何解救路参数或查询的改变并不会触发进入/离开的导航守卫？

#### 通过 watch 观察 `$route` 

```js
export default {
    watch: {
    "$route" (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

#### 使用`beforeRouteUpdate` 组件级守卫

```js
/*  
 * 在当前路由改变，但是该组件被复用时调用
 * 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
 * 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
 * 可以访问组件实例 `this`
*/
export default {
  beforeRouteUpdate(to, from, next) {
    /*进行你的逻辑操作*/
    next();
  }
}

```

**参考文档：**
[https://router.vuejs.org/zh/guide/advanced/navigation-guards.html](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%85%A8%E5%B1%80%E5%89%8D%E7%BD%AE%E5%AE%88%E5%8D%AB)
