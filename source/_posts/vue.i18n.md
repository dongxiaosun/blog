title: vue+element-ui+vue-i18n国际化中使用 cdn 性能优化详解
date: 2020-01-13
abbrlink: vue/element-i18n-cdn
categories:
  - 教程
tags:
  - vue
  - element-ui
  - vue-i18n
  - cdn
  - 性能优化

---

通过 cdn 优化代码性能时，一般情况下会将vue、element-ui、vue-i18n 等静态资源从依赖中抽离，抽取之后的国际化如何实现？

<!-- more -->

**项目依赖版本介绍**

+ vue-cli: `4.0.5`
+ vue: `2.6.10`
+ vue-router: `3.1.3` 
+ vue-i18n: `5.0.3`
+ element-ui: `2.12.0`

#### 编辑 public/index.html 文件

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  <title>demo</title>
  <!-- cdn start -->
  <script src="//unpkg.com/vue"></script>
  <script src="https://cdn.bootcss.com/vue-router/3.1.3/vue-router.min.js"></script>
  <script src="https://cdn.bootcss.com/vue-i18n/5.0.3/vue-i18n.min.js"></script>
  <link href="https://cdn.bootcss.com/element-ui/2.12.0/theme-chalk/index.css" rel="stylesheet">
  <script src="//unpkg.com/element-ui"></script>
  <script src="//unpkg.com/element-ui/lib/umd/locale/zh-CN.js"></script>
  <script src="//unpkg.com/element-ui/lib/umd/locale/en.js"></script>
  <!-- cdn end -->

  <script>
    // 中文
    Vue.config.lang = 'zh-cn'
    Vue.locale('zh-cn', ELEMENT.lang.zhCN)
    // 英文
    // Vue.config.lang = 'en'
    // Vue.locale('en', ELEMENT.lang.en)
  </script>
</head>

<body>
  <noscript>
    <strong>We're sorry but demo doesn't work properly without JavaScript enabled. Please enable it to
      continue.</strong>
  </noscript>
  <div id="app"></div>
  <!-- built files will be auto injected -->
</body>

</html>
```

#### 编辑 vue.config.js

**如果项目根目录下没有 vue.config.js（[详解](https://cli.vuejs.org/zh/config/#vue-config-js)） 文件，则创建 此文件**

通过 webpack 的 externals([详解](https://webpack.docschina.org/configuration/externals/)) 属，可以将引用的某一个库不让 webpack 打包。

```js
// vue.config.js 文件
module.exports = {
  configureWebpack: {
    externals: {
      // 以下是全局使用
      vue: "Vue",
      "element-ui": "ELEMENT",
      "vue-router": "VueRouter"
    }
  }
};
```

#### 处理项目业务中的国际化

以上配置仅仅配置了 element-ui 的国际化，但是在实际的开发中，我们也有可能会使用自己定义的一些变量采用国际化。

**示例:**

##### public/index.html 文件
```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  <title>demo</title>
    <!-- cdn start -->
  <script src="//unpkg.com/vue"></script>
  <script src="https://cdn.bootcss.com/vue-router/3.1.3/vue-router.min.js"></script>
  <script src="https://cdn.bootcss.com/vue-i18n/5.0.3/vue-i18n.min.js"></script>
  <link href="https://cdn.bootcss.com/element-ui/2.12.0/theme-chalk/index.css" rel="stylesheet">
  <script src="//unpkg.com/element-ui"></script>
  <script src="//unpkg.com/element-ui/lib/umd/locale/zh-CN.js"></script>
  <script src="//unpkg.com/element-ui/lib/umd/locale/en.js"></script>
    <!-- cdn end -->

  <!-- 系统自有语言包 -->
  <script src="lang/en.js"></script>
  <script src="lang/zh-CN.js"></script>

  <script>
    if (typeof Object.assign != 'function') {
      Object.assign = function (target) {
        'use strict';
        if (target == null) {
          throw new TypeError('Cannot convert undefined or null to object');
        }

        target = Object(target);
        for (var index = 1; index < arguments.length; index++) {
          var source = arguments[index];
          if (source != null) {
            for (var key in source) {
              if (Object.prototype.hasOwnProperty.call(source, key)) {
                target[key] = source[key];
              }
            }
          }
        }
        return target;
      };
    }
    /** 
    * 通过 Object.assign({}, COMMON.lang.en, ELEMENT.lang.en) 将 element-ui 和
    * 系统自有的语言包合并
    */
    /* 设置中文语言 */
    // Vue.config.lang = 'zh-cn'
    // Vue.locale('zh-cn', Object.assign({}, COMMON.lang.zhCN, ELEMENT.lang.zhCN));
    /* 设置英文语言 */
    Vue.config.lang = 'en'
    Vue.locale('en', Object.assign({}, COMMON.lang.en, ELEMENT.lang.en));
  </script>
</head>

<body>
  <noscript>
    <strong>We're sorry but demo doesn't work properly without JavaScript enabled. Please enable it to
      continue.</strong>
  </noscript>
  <div id="app"></div>
  <!-- built files will be auto injected -->
</body>

</html>

```

##### public/lang/en.js 文件
```js
// public/lang/en.js
(function(global, factory) {
  var mod = {
    exports: {}
  };
  factory(mod, mod.exports);
  global.COMMON = global.COMMON || {};
  global.COMMON.lang = global.COMMON.lang || {};
  global.COMMON.lang.en = mod.exports;
})(this, function(module, exports) {
  "use strict";

  exports.__esModule = true;
  exports.default = {
    global: {
      loading: "loading..."
    }
  };
  module.exports = exports["default"];
});
```

##### public/lang/zh-CN.js 文件
```js
// public/lang/zh-CN.js
(function(global, factory) {
  var mod = {
    exports: {}
  };
  factory(mod, mod.exports);
  global.COMMON = global.COMMON || {};
  global.COMMON.lang = global.COMMON.lang || {};
  global.COMMON.lang.zhCN = mod.exports;
})(this, function(module, exports) {
  "use strict";

  exports.__esModule = true;
  exports.default = {
    global: {
      loading: "正在加载中..."
    }
  };
  module.exports = exports["default"];
});
```

##### app.vue 文件
```html
<!-- app.vue -->
<template>
  <div id="app">
    <!-- $t("global.loading") 是 i18n 的写法-->
    {{ $t("global.loading") }}
  </div>
</template>
```


**注意：**
element-ui 中叙述的“通过 CDN 的方式加载语言文件”放方式有问题，element-ui官网代码如下：

```js
/**
 * 这里的代码运行会报错
 *
 * Vue.locale is not a function
 *
 * 这是因为 vue-i18n 的版本使用错误，应该使用 5.X(推荐5.0.3)版本
 */
<script src="//unpkg.com/vue"></script>
<script src="//unpkg.com/vue-i18n/dist/vue-i18n.js"></script>
<script src="//unpkg.com/element-ui"></script>
<script src="//unpkg.com/element-ui/lib/umd/locale/zh-CN.js"></script>
<script src="//unpkg.com/element-ui/lib/umd/locale/en.js"></script>

<script>
  Vue.locale('en', ELEMENT.lang.en)
  Vue.locale('zh-cn', ELEMENT.lang.zhCN)
</script>
```
