---
title: 简易版Vue-Router实现
date: 2020-03-19 20:19:20
tags: Vue
categories: Vue
---
## 简易版Vue-Router实现

​	Vue-Router重要性不用说了。不想当一个只会使用的coder，尝试实现一个简易版的Vue-Router，来帮助自己加深理解掌握Vue-Router。Vue-Router有两种模式。默认模式hash，也就是会变更url中#后面的内容，不会刷新浏览器，另一种个是history模式，history模式一般配合服务端的返回来使用。本次实现history模式的Vue-Router（hash模式也是异曲同工），且适用于SPA，不依赖服务端的返回。

### 前置知识

**插件、slot 插槽、混入、render 函数。**

前置知识不清楚建议先阅读Vue官方文档，可以帮助更好理解。



### Vue Router 的核心代码

```js
// 注册插件
// Vue.use() 内部调用传入对象的 install 方法
Vue.use(VueRouter)

// 创建路由对象
const router = new VueRouter({
routes: [
{ name: 'home', path: '/', component: homeComponent }
]
})
// 创建 Vue 实例，注册 router 对象
new Vue({
router,
render: h => h(App)
}).$mount('#app')
```



### 实现思路

- 创建 VueRouter 插件，静态方法 install  
    + 判断插件是否已经被加载  
    + 当 Vue 加载的时候把传入的 router 对象挂载到 Vue 实例上（注意：只执行一次）  
- 创建 VueRouter 类  
    + 初始化，options、routeMap、data (创建 Vue 实例作为响应式数据记录当前路
        径)  
    + initRouteMap() 遍历所有路由信息，把组件和路由的映射记录到 routeMap 对象中  
    + 注册 popstate 事件，当路由地址发生变化，重新记录当前的路径  
    + 创建 router-link 和 router-view 组件  
    + 当路径改变的时候通过当前路径在 routerMap 对象中找到对应的组件，渲染 router-view  



### 代码实现

```js
let _Vue = null
export default class VueRouter {
  // 接受两个参数，一个是Vue的构造函数
  static install (Vue) {
    if (VueRouter.install.installed) {
      return
    }
    // 1. 判断当前插件是否安装
    VueRouter.install.installed = true
    // 2. Vue构造函数记录到全局，后续使用
    _Vue = Vue
    // 3. 把创建Vue实例时传入的router对象注入到Vue实例
    // TODO 混入 所有Vue实例以及组件上都会被混入
    _Vue.mixin({
      beforeCreate () {
        // 只有当前Vue实例上具有$options,才在Vue原型上挂载$router对象
        if (this.$options.router) {
          _Vue.prototype.$router = this.$options.router
          // 进行初始化
          this.$options.router.init()
        }
      }
    })
  }

  // 构造函数
  constructor (options) {
    this.options = options
    this.routeMap = {}
    // observable 方法让一个对象可响应 Vue 内部会用它来处理 data 函数返回的对象
    // 返回的对象可以直接用于渲染函数和计算属性内，并且会在发生变更时触发相应的更新
    this.data = _Vue.observable({
      current: '/'
    })
  }

  // 初始化操作
  init () {
    this.createRouteMap()
    this.initComponent(_Vue)
    this.intiEvent()
  }

  // 将传递给Vue-Router对象的options对象转换成routeMap
  createRouteMap () {
    this.options.routes.forEach(route => {
      this.routeMap[route.path] = route.component
    })
  }

  // 创建router-link
  initComponent (Vue) {
    Vue.component('router-link', {
      props: {
        to: String
      },
      methods: {
        clickHandler (e) {
          // pushState方法改变浏览器的地址栏，会被浏览器的历史记住 不刷新页面，不向服务器发送请求
          window.history.pushState({}, '', this.to)
          // router-link是Vue实例 都可以访问Vue.prototype 注册了$router对象
          this.$router.data.current = this.to
          // 阻止默认事件行为
          e.preventDefault()
        }
      },
      // Vue默认创构建的是运行时版本，不支持template，可以通过vue-cli配置或者使用render函数
      // template: `<a :href="to"><slot></slot></a>`
      render (h) {
        // h 函数创建虚拟DOM 第一个参数标签名，第二个参数是属性对象， 第三个参数是内容
        return h('a', {
          attrs: {
            'href': this.to
          },
          on: {
            click: this.clickHandler
          }
        }, [this.$slots.default])
      }
    })

    const self = this
    // 创建router-view
    Vue.component('router-view', {
      render (h) {
        // 获取路由组件
        const component = self.routeMap[self.data.current]
        return h(component)
      }
    })
  }

  // 初始化事件处理
  intiEvent () {
    // popstate 事件函数 当浏览器地址栏发生变化时触发
    window.addEventListener('popstate', () => {
      // 改变当前的路径 data 是一个响应对象发生变化后重新加载组件
      this.data.current = window.location.pathname
    })
  }
}
```



### 注意点

1. 创建router-link和router-view组件时，没有使用template模板，因为vue-cli创建的项目默认使用的运行时版本的Vue。不支持template。解决方法如下

    - 如果想切换成带编译器版本的 Vue.js  。项目根目录创建 vue.config.js 文件，添加 runtimeCompiler  

    ```js
    module.exports = {
    runtimeCompiler: true
    }
    ```

    -  可以实现代码一样，不使用template，也不需要配置vue-cli，使用运行时版本支持的render函数对模板进行编译