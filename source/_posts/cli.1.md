---
title: 手写一个脚手架（一） # 文章标题
date: 2020-03-01  # 创建时间
abbrlink: cli # 文章名称
categories:
  - 教程 # 分类
tags:
  - cli # 标题
---

通过本教程的学习，自己可以手写一个脚手架，根据模板生成自己想要的项目（包括不局限于 vue、react，可以自行发挥），本教程是针对 vue 来写的。

<!-- more -->

在手写脚手架之前，我们需要做一些准备工作，学习一下基础知识。

## 什么是 cli

命令行界面（英语：command line interface，缩写：CLI）是在图形用户界面得到普及之前使用最为广泛的用户界面，它通常不支持鼠标，用户通过键盘输入指令，计算机接收到指令后，予以执行。

## npm package.json bin字段

在安装第三方带有bin字段的npm包时，可执行文件会被链接到当前项目的`./node_modules/.bin`中。在本项目中，就可以很方便地利用npm执行脚本（package.json文件中scripts可以直接执行：`node node_modules/.bin/practice`）；

+ 若是局部安装，则npm会为bin中配置的文件在bin目录下创建一个软连接（对于windows系统，默认会在`C:\Users\username\AppData\Roaming\npm`目录下）
+ 若是局部安装，则会在项目内的`./node_modules/.bin/`目录下创建一个软链接
  
因此，按上面的例子，当你安装`practice`的时候，npm就会为`./bin/www`在`/usr/local/bin/practice`路径创建一个软链接
如果你的模块只有一个可执行文件，并且它的命令名称和模块名称一样，你可以只写一个字符串来代替上面那种配置，例如：

`package.json`文件
```json
{ 
  "bin" : "./bin/www"
}
```

作用和如下写法相同:

`package.json`文件
```json
{
  "bin" : { 
    "practice" : "./bin/www"
  }
}
```

未完待续...

下期将介绍在开发脚手架中使用的一些npm包


__参考资料：__

[npm package.json属性详解](https://www.cnblogs.com/tzyy/p/5193811.html#_h1_11)
