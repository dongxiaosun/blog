title: Vue组件通信(一)
date: 2019-04-01
categories:
  - 教程
tags:
  - Vue

---

Vue 中组件关系可分为父子组件通信、兄弟组件通信、跨级组件通信。本节将介绍 "prop"、"$emit、$on"、 "中央事件总线(bus)"、"$parent、$children"的使用方式。

<!-- more -->

### 组件之间通信可以用下图表示:
![](/assets/component.jpg)
<font size="1"><center>组件通信示例</center></font>

### 下面将介绍组件之间通信的实现方式

## 1. prop

#### 父组件可以使用 props 把数据传给子组件

将父组件的数据 myMessage 通过设置标签 my-child 的 message 属性传递给子组件，子组件通过 props 传递接受，在子组件内 message 就是 父组件的 myMessage 数据。

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
  <div id="app">
    <my-child :message="myMessage"></my-child>
  </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<script>
	// 注册组件
	Vue.component("my-child", {
    props: ["message"],
    template: "<div>{{ message }}</div>"
	})

	var vm = new Vue({
		el: "#app",
		data() {
			return {
				myMessage: "hello world"
			}
		}
	})
</script>
</html>
```

**注意：**

某些组件在使用的时候，对传入的数据有类型要求。因此，我们可以为组件的prop 指定验证要求。可以为 props 中的值提供一个带有验证需求的对象，而不是一个字符串数组。例如：

```js
Vue.component("my-child", {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    message: Number,
    // 多个可能的类型
    message_1: [String, Number],
    // 必填的字符串
    message_2: {
      type: String,
      required: true // 是否必填
    },
    // 带有默认值的数字
    message_3: {
      type: Number,
      default: 100 // 默认值
    },
    // 带有默认值的对象
    message_4: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { name: "尼古拉斯赵四" }
      }
    },
    // 自定义验证函数
    message_5: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        // 返回布尔值来判断是否通过验证
        return ["success", "warning", "danger"].indexOf(value) !== -1
      }
    }
  },
  template: "<div>{{ message }}</div>"
})
```


## 2. $emit、$on

#### 子组件可以使用 $emit 触发父组件的自定义事件

```yml
vm.$emit(event, arg); // 触发当前实例上的事件
vm.$on(event, fn); // 监听event事件后运行 fn
```

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
  <div id="app">
    <div> 数量: {{ total }} </div>
    <my-child @update="parentUpdate"></my-child>
  </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<script>
	// 注册组件
	Vue.component("my-child", {
    template: '<button @click="childUpdate">+1</button>',
		methods: {
			childUpdate() {
				// 在子组件中通过 $emit 将事件和数据发射出去
				// 父组件通过 $on 接受子组件的事件和数据

				// update 事件名， 1数据
				this.$emit("update", 1);
			}
		}
	})

	var vm = new Vue({
		el: "#app",
		data: function () {
			return {
				total: 1
			}
		},
		methods: {
			parentUpdate(val) {
				// val 子组件传递的数据
				this.total += 1;
			}
		}
	})
</script>
</html>
```

## 3. bus (使用一个空的 Vue 实例作为中央事件总线)

中央事件总线可以实现各种组件之间通信，并且可以在中央事件总线中设置数据，实现数据共享

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
  <div id="app">
		<div> 姓名： {{ name }}</div>
    <div> 年龄: {{ age }}</div>
    <my-child></my-child>
  </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<script>
	// 创建一个空的 Vue 实例，作为中央事件总线
	// 同时也可以在中央事件总线中定义全局的数据
	var bus = new Vue({
		data() {
			return {
				name: "大東"
			}
		}
	});

	// 注册组件
	Vue.component("my-child", {
    template: '<button @click="childUpdate">+1</button>',
		methods: {
			childUpdate() {
				bus.name = "大東博客";
				// update 为事件名， 1为数据
				bus.$emit("update", 1);
			}
		}
	})

	var vm = new Vue({
		el: "#app",
		data: function () {
			return {
				age: 1
			}
		},
		computed: {
			name() {
				return bus.name;
			}
		},
		mounted: function() {
			var _this = this;
			bus.$on("update", function(val) {
				_this.age += val;
			});
		}
	})
</script>
</html>

```

## 4. $parent、$children

在子组件中， 使用 this.$parent 可以直接访问改组件的父实例或组件，父组件也可以通过 this.$children 访问它所有的子组件，而且可以递归向上或向下无限访问，直到根实例或最内层的组件。

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
  <div id="app">
		<div> 姓名： {{ name }}</div>
    <div> 年龄: {{ age }}</div>
    <my-child></my-child>
  </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<script>

	// 注册组件
	Vue.component("my-child", {
    template: '<button @click="childUpdate">+1</button>',
		methods: {
			childUpdate() {
				// 通过 $parent 可以访问父组件的方法
				this.$parent.updateAge();
			}
		}
	})

	var vm = new Vue({
		el: "#app",
		data: function () {
			return {
				age: 1,
				name: "大東博客"
			}
		},
		methods: {
			updateAge: function() {
				this.age += 1;
			}
		}
	})
</script>
</html>
```

**注意：**
在父组件模板中，子组件标签上使用 ref 指定一个名称，并在父组件内通过 this.$refs 来访问指定名称的子组件。




