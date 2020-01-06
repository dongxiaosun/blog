title: async...await 优雅的处理错误方法
date: 2019-10-09
categories:
  - 学习
tags:
  - ES2017

---


在使用 async...await 方法时，经常采用 try...catch 捕获异常，如果有多个异步操作，需要每一次书写 try...catch。这样代码的简洁性较差，为了使代码更加的优雅，我们通过使用 `await-to-js` js 库来处理异常。

<!-- more -->

## 常规写法

_代码简洁性较差_

```js
let flag = true;

function fetchVehicle() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (flag) {
        resolve(true);
      } else {
        reject(false);
      }
    }, 1000);
  });
}

function fetchOrganization() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (flag) {
        resolve(true);
      } else {
        reject(false);
      }
    }, 2000);
  });
}

async function fn() {
  // 每个异步都需要处理
  try {
    let res = await fetchVehicle();
    console.log(res);
  } catch (error) {
    console.log(error);
  }

  try {
    let res = await fetchOrganization();
    console.log(res);
  } catch (error) {
    console.log(error);
  }
}

fn();
```

## await-to-js 优雅写法

### 核心思路

抽离一个公共方法，将 try...catch 抽离

```js
let flag = true;

function fetchVehicle() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (flag) {
        resolve(true);
      } else {
        reject(false);
      }
    }, 1000);
  });
}

/* await-to-js 代码思路
这是因为 asycn...await 的本质就是 `promise` 的语法糖
因此可以使用 then 函数 */

async function fn() {
  // 每个异步都需要处理
  let [error, res] = await fetchVehicle()
    .then(res => [null, res])
    .catch(error => [error]);

  console.log("error", error);
  console.log("res", res);
}

fn();
```

### 封装方法

```js
/* 方法抽取 */
const to = promise => {
  return promise.then(res => [null, res]).catch(error => [error]);
};

let flag = true;

function fetchVehicle() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (flag) {
        resolve(true);
      } else {
        reject(false);
      }
    }, 1000);
  });
}

async function fn() {
  // 每个异步都需要处理
  let [error, res] = await to(fetchVehicle())
  console.log("error", error);
  console.log("res", res);
}

fn();


```

[await-to-js 源码参考](https://github.com/scopsy/await-to-js)
