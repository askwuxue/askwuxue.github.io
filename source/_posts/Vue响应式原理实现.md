---
title: 学习Vue.js 响应式原理
date: 2021-01-10 21:01:11
tags: ["Vue"]
categories: Vue
---

## 学习Vue.js 响应式原理

_最近在学习Vue的源码，对于核心原理比较感兴趣。本文就Vue.js的响应式原理进行模拟实现。代码实现不难，花二十分钟看一下，`download`到本地`run`一下。你也会有很大收获。加油！！！_

### 前置知识

- 数据驱动
- 响应式的核心原理
- 发布订阅模式和观察者模式  

#### 数据驱动

- 数据响应式  
    - 数据模型仅仅是普通的 JavaScript 对象，而当我们修改数据时，视图会进行更新，避免了繁
        琐的 DOM 操作，提高开发效率  
- 双向绑定  
    - 数据改变，视图改变；视图改变，数据也随之改变  
    - 我们可以使用 v-model 在表单元素上创建双向数据绑定  
- 数据驱动
    - 开发过程中仅需要关注数据本身，不需要关心数据是如何渲染到视图 

#### 数据响应式的原理 

​	因为Vue3相较于Vue2原理发生了变化，本文讨论Vue2。Vue3将在下一篇中分析

**Vue2.X**

```js
// 模拟Vue实例
const vm = {} 
// 模拟Vue中的data选项
const data = {
    message: 'Hello World',
    count: 1
}
const ProxyData = () => {
    Object.keys(data).forEach((key) => {
        Object.defineProperty(vm, key, {
            enumerable: true,
            configurable: true,
            get () {
                return data[key]
            },

            set (newValue) {
                if (newValue === data[key]) {
                    return newValue
                }
                data[key] = newValue
                document.querySelector('#app').textContent = newValue
            }
        })
    })
}
```



#### 发布订阅模式和观察者模式  

##### 发布/订阅模式

- 订阅者
- 发布者
- 信号中心

> 假设存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信
> 号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执
> 行。这就叫做"发布/订阅模式"（publish-subscribe pattern）  

```js
// 事件中心
class EventEimt {
  constructor () {
    this.subs = Object.create(null)
  }

  // 注册事件 发布事件
  $on (eventType, handler) {
    this.subs[eventType] = this.subs[eventType] ? this.subs[eventType] : []
    this.subs[eventType].push(handler)
  }

  // 触发事件 订阅事件
  $emit (eventType) {
    this.subs[eventType]?.forEach(handler => handler())
  }
}

// 测试用例
const ev = new EventEimt()

ev.$on('click', () => {
  console.log('click1')
})

ev.$on('click', () => {
  console.log('click2')
})

ev.$emit('c')
```



##### 观察者模式 

- 观察者(订阅者) -- Watcher

    - update()：当事件发生时，具体要做的事情
- 目标(发布者) -- Dep
    - subs 数组：存储所有的观察者
    - addSub()：添加观察者
    - notify()：当事件发生，调用所有观察者的 update() 方法
    - 没有事件中心  

```js
// 发布者
class Dep {
  constructor () {
    // 记录所有订阅者
    this.subs = []
  }

  // 添加订阅者
  addSub (sub) {
    if (sub && sub.update) {
      this.subs.push(sub)
    }
  }

  // 通知订阅者
  notify () {
    this.subs.forEach(sub => sub.update())
  }
}

// 订阅者
class Watch {
  update () {
    console.log('update')
  }
}

// 测试用例
const pub = new Dep()
const w1 = new Watch()
pub.addSub(w1)
pub.addSub(w1)
pub.notify()
```



### Vue 响应式原理

#### 整体架构图

![Vue 响应式原理](https://github.com/askwuxue/FrontEnd/blob/master/note-images/vue-2.png)

上图中各部分的功能以及实现如下：

#### Vue

- 负责接收初始化的参数(选项)
- 负责把 data 中的属性注入到 Vue 实例，转换成 getter/setter
- 负责调用 observer 监听 data 中所有属性的变化
- 负责调用 compiler 解析指令/插值表达式  

```js
class Vue {
  constructor (options = {}) {
    // 1. 保存创建Vue实例时传递的options
    this.$options = options
    this.$data = options.data || {}
    // options.el是选择器或者DOM对象
    this.$el = typeof options.el === 'string' ? document.querySelector(options.el) : options.el
    // 2. 把Data中的成员转换成get和set
    this._ProxyData(this.$data)
    // 3. observer对象，监听data数据的变化
    new Observer(this.$data)
    // 4. 调用compile函数，解析指令和差值表达式
    new Compiler(this)
  }

  // 注册数据的getter和setter方法
  _ProxyData (data) {
    Object.keys(data).forEach(key => {
      Object.defineProperty(this, key, {
        configurable: true,
        enumerable: true,
        get () {
          return data[key]
        },
        set (newValue) {
          if (newValue === data[key]) {
            return
          }
          data[key] = newValue
        }
      })
    })
  }
}
```

#### Observer

- 负责把 data 选项中的属性转换成响应式数据
- 如果data 中的某个属性是对象，把该属性转换成响应式数据
- 数据变化发送通知

```js
class Observer {
  constructor (data) {
    this.walk(data)
  }

  // 遍历data属性
  walk (data) {
    // data不存在或者data不是对象
    if (!data || typeof data !== 'object') {
      return
    }
    // 遍历data对象
    Object.keys(data).forEach(key => this.defineReactive(data, key, data[key]))
  }

  // 为data对象上的所有属性注册get和set
  defineReactive (obj, key, val) {
    const that = this
    // 负责收集依赖并发布通知
    let dep = new Dep()
    // TODO data中的属性值可能是对象类型，调用walk方法，如果是对象类型，可以递归的将其设置为响应式数据
    this.walk(val)
    Object.defineProperty(obj, key, {
      configurable: true,
      enumerable: true,
      // 监听对$data数据的访问
      get () {
        // 收集依赖
        Dep.target && dep.addSub(Dep.target)
        // TODO 此处只能使用val值而不能使用obj[key] 因为只要访问data的数据就会调用observer对象的get方法。会导致堆栈溢出
       	return val
      },
      // 监听对$data对象的改变
      set (newValue) {
        if (newValue === val) {
          return
        }
        val = newValue
        // TODO 如果为data中的数据进行了重新赋值为对象，那么需要对该对象遍历，注册get和set
        // 此处的this指向了data对象，不是observer对象
        that.walk(newValue)
        // 发送通知
        dep.notify()
      }
    })
  }
}
```



#### Compiler

- 负责编译模板，解析指令/插值表达式
- 负责页面的首次渲染
- 当数据变化后重新渲染视图

```js
class Compiler {
  // 接收vue实例
  constructor (vm) {
    this.el = vm.$el
    this.vm = vm
    this.compile(vm.$el)
  }

  // 编译模板 处理文本节点和元素节点
  compile (el) {
    // 遍历el节点的子节点
    Array.from(el.childNodes).forEach(node => {
      // 子节点为元素节点
      if (this.isElementNode(node)) {
        this.compileElement(node)
      }
      // 子节点为文本节点
      if (this.isTextNode(node)) {
        this.compileText(node)
      }
      // 子节点还存在子节点
      if (node.childNodes && node.childNodes.length) {
        this.compile(node)
      }
    })
  }

  // 编译元素节点，处理指令
  compileElement (node) {
    console.log('node: ', node.attributes);
    Array.from(node.attributes).forEach(attr => {
      // 判断是否是指令
      if (this.isDirective(attr.name)) {
        // 指令的名
        const attrName = attr.name.substr(2)
        // 指令的值，即数data的key值
        const key = attr.value
        this.update(node, key, attrName)
      }
    })
  }

  // 统一更新
  update (node, key, attrName) {
    const updateFn = this[attrName + 'Updater']
    // 调用指令处理函数并且传递data数据
    updateFn && updateFn(node, this.vm[key])
  }

  // 处理v-text指令
  textUpdater (node, value) {
    node.textContent = value
  }

  // 处理v-model指令
  // TODO 没有实现数据的双向绑定
  modelUpdater (node, value) {
    node.value = value
  }

  // 编译文本节点，处理插值表达式
  compileText (node) {
    const reg = /\{\{(.*?)\}\}/g
    reg.exec(node.textContent)
    node.textContent = node.textContent.replace(reg, (match, key) => this.vm[key.trim()])
  }

  // 判断元素属性是否为指令
  isDirective (attrName) {
    return attrName.startsWith('v-')
  }

  // 判断节点是否是文本节点
  isTextNode (node) {
    return node.nodeType === 3
  }

  // 判断节点是否为元素节点
  isElementNode (node) {
    return node.nodeType === 1
  }
}
```





- Dep
    - 收集依赖，添加观察者(watcher)
    - 通知所有观察者

```js
class Dep {
  constructor () {
    // 存储所有观察者
    this.subs = []
  }

  // 添加观察者
  addSub (sub) {
    if (sub && sub.update)
    this.subs.push(sub)
  }

  // 发送通知
  notify () {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
```



- Watcher
    - 当数据变化触发依赖， dep 通知所有的 Watcher 实例更新视图
    - 自身实例化的时候往 dep 对象中添加自己

```js
class Watch {
  constructor (vm, key, cb) {
    this.vm = vm
    // data属性
    this.key = key
    // 更新视图的回调函数
    this.cb = cb
    // watch对象记录到Dep的静态方法target上
    Dep.target = this
    // 触发get方法，触发Dep的addSub方法
    this.oldValue = vm[key]
    Dep.target = null
  }
  update () {
    let newValue = this.vm[this.key]
    if (newValue === this.oldValue) {
      return
    }
    // 更新视图
    this.cb(newValue)
  }
}
```



### 测试

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app">
    <div>{{count}}</div>
    <div>{{message}}</div>
    <p>v-text: </p>
    <div v-text="obj"></div>
    <!-- TODO 有一个bug 差值表达式有空格无法正常渲染，data发生变化时，全部覆盖了 -->
    <p>v-model: {{input}}</p>
    <input type="text" v-model="input">
  </div>
  <script src="./dep.js"></script>
  <script src="./watch.js"></script>
  <script src="./vue.js"></script>
  <script src="./observer.js"></script>
  <script src="./compiler.js"></script>
  <script>
    const vm = new Vue({
      el: '#app',
      data: {
        count: 1,
        message: 'hello',
        obj: '<p>hello</p>',
        input: '222'
      }
    })
  </script>
</body>
</html>
```

### 总结

​	因为Vue.js的功能丰富，一个一个功能去实现也没有必要。本文只是抱着学习的心态，实现了Vue.js的几个核心功能。大概是两百行左右的代码，整体来说核心功能的实现代码不是很难，但是思想值得学习。如果你也对Vue.js的实现原理感兴趣的话，也可以访问我的[github](https://github.com/askwuxue/note_source/tree/master/vue/vue-core/mini-vue2)，clone到本次run一下，debugger一下。可以发现我的实现有bug的地方。也可以看一下我的实现方式，代码有超多注释，通俗易懂哈哈哈。

