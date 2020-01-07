<!--
 * @Description: 
 * @Author: sunxiaodong
 * @Date: 2020-01-02 19:23:41
 * @LastEditors: sunxiaodong
 * @LastEditTime: 2020-01-07 08:44:12
 -->
title: Vue组件通信(二)
date: 2019-04-23
categories:
  - 教程
tags:
  - Vue

---

Vue 中组件关系可分为父子组件通信、兄弟组件通信、跨级组件通信。本节将介绍 "provide/inject"、"Vuex"的使用方式。

<!-- more -->

### 组件之间通信可以用下图表示:
![](/assets/image/component.jpg)
<font size="1"><center>组件通信示例</center></font>

### 下面将介绍组件之间通信的实现方式

## 1. provide/inject

provide 和 inject 主要为高阶插件/组件库提供用例，并不推荐直接用于应用程序代码中。
允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div class="app" id="app">
    <my-parent></my-parent>
  </div>
  <script src="./vue.js"></script>
  <script>
    // 子组件
    Vue.component("my-children", {
      inject: {
        userName: {
          from: "name", // 指的是上游组件 provide 中的属性名称(name)
          default: "华仔" // 设置默认值
        }
      },
      template: "<h2>{{userName}}</h2>"
    })

    // 父组件
    Vue.component("my-parent", {
      template: "<div><my-children></my-children></div>"
    })

    var vm = new Vue({
      el: "#app",
      provide: {
        name: "刘德华"
      },
      data: {
        total: 0,
        timer: null
      }
    })
  </script>
</body>
</html>
```

## Vuex
Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。

![](/assets/image/vuex.png)
<font size="1"><center>Vuex 原理图</center></font>

详细资料参考官网:
[https://vuex.vuejs.org/zh](https://vuex.vuejs.org/zh)