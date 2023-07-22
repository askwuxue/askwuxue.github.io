---
title: keep-alive实现页面缓存
date: 2021-02-09 09:31:30
tags: Vue
categories: Vue
---

#### 1. 业务场景

​	现在有A,B,两个页面，在B页面做一些操作，比如说通过输入搜索条件搜出相关的数据，然后点击一条数据，跳转到详情页A，在A页面点击返回按钮，B页面还保持离开前的状态，但是从B页面到A页面，A页面一直是最新的页面。

#### 2. 具体做法

##### keep-alive使用

​	**注：**对于这个例子来说，我会将A页面和B页面都缓存。实际开发中，选择自己的要缓存的组件就可以，我只是为了演示这个例子。

```vue
<template>
  <div id="app">
    <nav>
      <router-link to="/">to A page</router-link>
      <router-link to="/about">to B page</router-link>
    </nav>
    <!-- 需要缓存的视图 -->
    <keep-alive>
      <router-view />
    </keep-alive>
  </div>
</template>
```



##### `keep-alive`的生命周期

- 初次进入时：
  1. `created` > `mounted` > `activated`
  2. 退出后触发 `deactivated`
- 再次进入：
  1. 只会触发 `activated`
- 再次离开：
  1. 退出后触发 `deactivated`

了解了keep-alive的钩子函数后，结合业务，我们要在离开B页面时保存当前的位置。不用想，这个时候必须要监听`scroll`事件，什么时候监听呢？因为每次进入B页面，都可能会滚动页面，所以在`activated`钩子函数中做这件事无疑是最合适的。如下

```vue
<script>
export default {
  name: 'AboutView',
  components: {},
  data() {
    return {
      scrollTop: '',
    }
  },
  // 进入当前组件时监听scroll事件，并且将页面位置记录到data对象scrollTop中。
  // 根据保存的scrollTop值回到上一次离开页面的位置
  activated() {
    console.log('About activated ...')
    document.documentElement.scrollTop = this.scrollTop
    window.addEventListener('scroll', this.handlerScroll)
  },
  // 离开页面，移除scroll事件监听
  deactivated() {
    console.log('About deactivated ...')
    window.removeEventListener('scroll', this.handlerScroll)
  },
  methods: {
    handlerScroll() {
      this.scrollTop = document.documentElement.scrollTop
    },
  },
}
</script>
```

如上所示，当离开B页面时，记录了当前的位置，当再次进入时，根据之前保存的数据。回到离开前的位置。

#### 3. 有哪些可以优化的点？

​	可以注意到上面我们注册了`scroll`滚动事件，这种事件触发的频率比较高，一般会使用防抖或者节流来进行限制。

```vue
<script>
const _ = require('lodash')
export default {
  name: 'AboutView',
  components: {},
  data() {
    return {
      scrollTop: '',
      removeFlag: null,
    }
  },
  // 进入当前组件时监听scroll事件，并且将页面位置记录到data对象scrollTop中。
  // 根据保存的scrollTop值回到上一次离开页面的位置
  activated() {
    console.log('About activated ...')
    document.documentElement.scrollTop = this.scrollTop
    window.addEventListener('scroll', this.handlerDebounceScroll())
  },
  // 离开页面，移除scroll事件监听
  deactivated() {
    console.log('About deactivated ...')
    window.removeEventListener('scroll', this.handlerDebounceScroll())
  },
  methods: {
    handlerScroll() {
      this.scrollTop = document.documentElement.scrollTop
    },
    handlerDebounceScroll(delay = 500) {
      // 移除事件的标识
      if (!this.removeFlag) {
        this.removeFlag = _.debounce(this.handlerScroll, delay)
      }
      return this.removeFlag
    },
  },
}
</script>
```

