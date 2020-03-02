---
title: 手写一个脚手架（二） # 文章标题
date: 2020-03-04 # 创建时间
abbrlink: cli-2 # 文章名称
categories:
  - 教程 # 分类
tags:
  - cli # 标题
---

通过本教程的学习，自己可以手写一个脚手架，根据模板生成自己想要的项目（包括不局限于 vue、react，可以自行发挥），本教程是针对 vue 来写的。

<!-- more -->

今天给大家介绍开发脚手架中使用的 npm 包：

```
- commander
- ora
- inquirer
- util{ promisify }
- fs-extra
- metalsmith
- consolidate
- download-git-repo
```

## npm 包介绍

### commander
commander是node.js命令行界面的完整解决方案。该包主要用来设置程序的命令，例如：vue create XXX 类似命令。

详细教程：[https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md](https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md)

### ora
ora包用于显示加载中的效果，类似于前端页面的loading效果。

详细教程：[https://www.npmjs.com/package/ora](https://www.npmjs.com/package/ora)

### inquirer
一个用户与命令行交互的工具。开始通过npm init创建package.json的时候就有大量与用户的交互(当然也可以通过参数来忽略输入)；而现在大多数工程都是通过脚手架来创建的，使用脚手架的时候最明显的就是与命令行的交互，如果想自己做一个脚手架或者在某些时候要与用户进行交互，这个时候就不得不提到inquirer.js了。

详细教程：[https://blog.csdn.net/qq_26733915/article/details/80461257](https://blog.csdn.net/qq_26733915/article/details/80461257)

### util{ promisify }

### fs-extra

### metalsmith

### consolidate

### download-git-repo

__参考资料：__

[commander.js](https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md)