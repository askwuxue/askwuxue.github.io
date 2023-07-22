---
title: React学习笔记
date: 2020-04-14 20:13:53
tags: React
categories: React
---

# React学习笔记
## 1. 什么时候需要使用类组件的构造函数
根据官方给出的实例，大概只有这么三种情况下，才必须要使用类组件的构造函数。
1. 需要在构造函数中初始化state。
2. 必须在构造函数中对this.props进行接收(通常来说这种情况不是必须的)。
3. 需要绑定类组件中自定义方法的this。(因为自定义方法中的this，如果不是使用箭头函数
定义的，则自定的函数中的this指向会丢失。所以需要在constructor中使用bind进行绑定。这种情况可以使用箭头函数来代替。)
```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
}
```
## 2. 如何实现组件的通信

### 2.1 自上而下的数据流
  React哲学是从上至下的数据流，当从上至下进行通信时，通常的方式是，上层组件传递给下层组件的数据放在下层组件的属性上，最终下层组件会在props对象上拿到上层组件传递的数据并进行渲染。
### 2.2 同级组件间的通信

#### 2.2.1 React原生
  React不支持同级组件间直接通信，提出了一种状态提升的概念。需要组件通信时，调用上层组件传递的方法，将需要传递的内容放在该函数的实参中。上层组件在该函数的定义体中拿到下层组件传递的数据，也就是状态提升到上层组件，上层组件将数据传递给其他子组件，从而实现了同级组件的通信。

#### 2.2.2 发布订阅
  当组件间的关系变得复杂时，react原生的组件通信方式，会产生很多"中间者"性质的`props`。代码会变得允许且不优雅。发布订阅模式是其中的一个解决方案。
  发布订阅，即，需要发起通信的组件，发布消息，需要接受消息的组件，订阅消息。当消息发布，接受消息的组件，知道消息发布，从而可以接收到消息。伪代码如下：

**订阅消息的组件**

```jsx
import React, { Component } from 'react'
import PubSub from 'pubsub-js'
export default class List extends Component {
	// 组件挂载结束之后订阅
    componentDidMount() {
        this.token = PubSub.subscribe('setData', (msg, data) => {
            // 根据订阅结果更新state
            this.setState(data);
        })
    }

	// 组件卸载了之后清除订阅
	componentWillUnmount() {
	    PubSub.unsubscribe(this.token);
	}
}
```


**发布消息的组件**

```js
import React, { Component } from 'react'
import PubSub from 'pubsub-js'

export default class Search extends Component {
	// 调用getUserInfo函数时，发布消息
	getUserInfo = () => {
	PubSub.publish('setData', {
                isFirst: false,
                isLoading: true,
                isError: false,
                listItems: [] 
            });
	}
}

```



#### 2.2.3 Redux

  使用发布订阅可以解决一些问题，但是随着项目的规模升级，复杂度升级。使用发布订阅，也可以解决问题，组件内到处都是发布订阅的方法，项目会变得越来越难以掌控，维护性也会变差。随之，`redux（状态管理）`这种解决方案出现了。和发布订阅一样，`redux`也不是原生React的内容，是官方维护的第三方库，所以可以放心使用。

> **"只有遇到 React 实在解决不了的问题，你才需要 Redux 。"**

**原理**

![](/img/React/redux.jpg)

图中，有几个概念还是需要清楚的。

1. **Store**

   Store，状态管理仓库，所以的状态在Store中进行管理。

2. **Action Creators**

   动作创造者，如 进行 + 1 的操作，就会创造出一个`action`, 形式是：{  type: '+'， data: 1 }

3. **Reducers**

   最终进行处理的地方

`Store`作为一个中心调度者，将`action`分发给`reducers`, `reducers`