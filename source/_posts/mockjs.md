title: Vue+Mock.js，模拟接口数据，实现前后端独立开发
date: 2019-03-21
categories:
  - 教程
tags:
  - Vue
  - Mock.js

---

前后端分工协作是一个非常高效的做法，但是有时前后端分离不彻底会很痛苦。前后端应该是异步进行的，进度互不影响，但是在没有数据的时候，前端却严重依赖后端的接口，总会苦苦等待后端接口出来才能继续开发。为了解决这个问题，大神就造了一个轮子，供大家使用--Mock.js。

<!-- more -->

## 关于 mock.js，官网描述：

> 1. 前后端分离
> 2. 不需要修改既有代码，就可以拦截 Ajax 请求，返回模拟的响应数据
> 3. 数据类型丰富
> 4. 通过随机数据，模拟各种场景

本文介绍在 vue 项目中如何使用 Mock.js
通过 vue-cli 搭建的项目（v2.9.3）

## 1. 安装

```yml
cnpm install mockjs --save-dev
```

## 2. 配置

为了只在开发环境使用 Mock.js，而打包到生产环境时自动不使用 Mock.js，做以下配置：
_config 目录下 dev.env.js_

```js
"use strict";

const merge = require("webpack-merge");
const prodEnv = require("./prod.env");

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  Mock: true
});
```

_config 目录下 prod.env.js_

```js
"use strict";

module.exports = {
  NODE_ENV: '"production"',
  Mock: false
};
```

_src 目录下 main.js_

```js
// 向main.js中添加如下代码
process.env.Mock && require("./mock.js");
```

## 3. 创建文件

_在 src 目录下创建 mock.js，内容如下_

```js
// 引入mockjs
const Mock = require("mockjs");
// 获取 mock.Random 对象
const Random = Mock.Random;
// mock一组数据
const produceNewsData = function() {
  let articles = [];
  for (let i = 0; i < 100; i++) {
    let newArticleObject = {
      title: Random.csentence(5, 30), //  Random.csentence( min, max )
      thumbnail_pic_s: Random.dataImage("300x250", "mock的图片"), // Random.dataImage( size, text ) 生成一段随机的 Base64 图片编码
      author_name: Random.cname(), // Random.cname() 随机生成一个常见的中文姓名
      date: Random.date() + " " + Random.time() // Random.date()指示生成的日期字符串的格式,默认为yyyy-MM-dd；Random.time() 返回一个随机的时间字符串
    };
    articles.push(newArticleObject);
  }
  return {
    data: articles
  };
};
// 拦截ajax请求，配置mock的数据
Mock.mock("/api/articles", "get", produceNewsData);
```

模板的功能非常强大，可以生成几乎所有类型的数据，具体参考:
官方文档：[https://github.com/nuysoft/Mock/wiki](https://github.com/nuysoft/Mock/wiki)

## 4. 页面使用

```js
export default {
  data() {
    return {
      articles: []
    };
  },
  methods: {
    requestData() {
      this.$axios
        .get("/api/articles")
        .then(response => {
          // 打印返回的数据
          console.log(response);
        })
        .catch(error => {
          console.log(error);
        });
    }
  },
  created() {
    this.requestData();
  }
};
```
