---
title: Promise实现原理
date: 2020-10-11 20:21:11
tags: JavaScript
categories: JavaScript
---
# Promise实现原理

本文旨在实现Promise，Promise用法请求移步至[阮一峰老师ES6教程](https://es6.ruanyifeng.com/#docs/promise)

### Promise用法

```js
const p1 = new Promise((resolve, reject) => {
  console.log('create a promise');
  resolve('成功了');
})

console.log("after new promise");

const p2 = p1.then(data => {
  console.log(data)
  throw new Error('失败了')
})

const p3 = p2.then(data => {
  console.log('success', data)
}, err => {
  console.log('faild', err)
})
```

输出

```js
"create a promise"
"after new promise"
"成功了"
"faild Error: 失败了"
```



### Promise基本特征

1. promise 有三个状态：`pending`，`fulfilled`， `rejected`；

2. `new promise`时， 需要传递一个`executor()`执行器，执行器立即执行；

3. `executor`接受两个参数，分别是`resolve`和`reject`；

4. promise  的默认状态是 `pending`；

5. promise 有一个`value`保存成功状态的值，可以是`undefined/thenable/promise`；

6. promise 有一个`reason`保存失败状态的原因；

7. promise 只能从`pending`到`rejected`, 或者从`pending`到`fulfilled`，状态一旦确认，就不会再改变；

8. promise 必须有一个`then`方法，then 接收两个参数，分别是 promise 成功的回调 successCallBack, 和 promise 失败的回调 failCallBack；

9. 如果调用 then 时，promise 已经成功，则执行`successCallBack`，参数是`promise`的`value`；

10. 如果调用 then 时，promise 已经失败，那么执行`failCallBack`, 参数是`promise`的`reason`；

11. 如果 then 中抛出了异常，那么就会把这个异常作为参数，传递给下一个 then 的失败的回调`failCallBack`；



### Promise实现

​	因为Promise实现有很多的细节，直接贴上最终版的代码，可能会劝退很多人，也不是很好理解。我们从最基础的Promise对象功能以及方法的实现入手，由简单到复杂，展现最终效果。

#### 1. 基础版Promise实现

​	如果大家了解了Promise的基本特征，很容易实现这个简易版的Promise对象。代码如下，有详细的注释。

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = undefined
  // 失败的回调函数
  failCallBack = undefined
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
  }
  // then
  then(successCallBack, failCallBack) {
    if (this.status === FULFILLED) {
      successCallBack(this.value)
    } else if (this.status === REJECTED) {
      failCallBack(this.reason)
    }
  }
}
// 导出Promise
module.exports = MyPromise
```

#### 2. 异步的Promise

​	上面的简易版中，我们基本实现了Promise的特征。但是少了Promise的灵魂，我们都知道Promise的出现，是为了解决异步调用的问题，但是简易版中我们并没有支持异步调用。实现之前，我们先看一下，Promise是如何支持异步调用的。

下面是一个`Promise`对象的简单例子。

```js
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

上面代码中，`timeout`方法返回一个`Promise`实例，表示一段时间以后才会发生的结果。过了指定的时间（`ms`参数）以后，`Promise`实例的状态变为`resolved`，就会触发`then`方法绑定的回调函数。

Promise 新建后就会立即执行。

```js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

上面代码中，Promise 新建后立即执行，所以首先输出的是`Promise`。然后，`then`方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以`resolved`最后输出。

​	现在请大家认真观察简易版中then方法的实现，我们只匹配了fulfilled的状态和rejected的状态，如果想要支持异步的调用，那么then方法还应该存在pending状态，当resolve或者reject方法触发时，then方法的pending状态才会发生变化。

##### 核心

- then方法pending状态时将成功回调和失败回调暂存
- 调用resolve方法时，判断是否存在successCallBack方法，存在则调用
- 调用reject方法时，判断是够存在failCallBack，存在则调用

##### 实现代码

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = undefined
  // 失败的回调函数
  failCallBack = undefined
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
    // 调用then方法中的回调函数
    this.successCallBack && this.successCallBack(value)
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
    // 调用then方法中的回调函数
    this.failCallBack && this.failCallBack(reason)
  }
  // then
  then(successCallBack, failCallBack) {
    if (this.status === FULFILLED) {
      successCallBack(this.value)
    } else if (this.status === REJECTED) {
      failCallBack(this.reason)
      // pending状态时将成功回调和失败回调暂存
    } else {
      this.successCallBack = successCallBack
      this.failCallBack = failCallBack
    }
  }
}

module.exports = MyPromise
```



#### 3. 多次调用then方法

​	因为then方法可以多次调用，我们要对多次then方法调用进行处理。如果是同步的情况下我们不需要进行特殊处理。因为每一个then会等待上一个then方法执行结束后执行，我们需要对异步情况下的then方法多次调用进行处理。处理逻辑其实很简单，我们只需要对then方法的pending逻辑进行修改就可以，一个then方法得到时候，我们只需要一个变量进行存储回调函数，多个then方法调用。我们只需要将回调函数存储到一个数组中。当执行resolve方法或者reject方法时，我们按照顺序将数组中的回调函数取出并执行就可以了。

##### 核心

- then方法的pending状态时，将回调函数存储到数组中。
- 调用resolve方法时，判断是否存在successCallBack方法，存在则调用
- 调用reject方法时，判断是够存在failCallBack，存在则调用

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = []
  // 失败的回调函数
  failCallBack = []
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
    // 调用then方法中的回调函数
    while (this.successCallBack.length) {
      this.successCallBack.shift()(this.value)
    }
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
    // 调用then方法中的回调函数
    while (this.failCallBack.length) {
      this.failCallBack.shift()(this.reason)
    }
  }
  // then
  then(successCallBack, failCallBack) {
    if (this.status === FULFILLED) {
      successCallBack(this.value)
    } else if (this.status === REJECTED) {
      failCallBack(this.reason)
      // pending状态时将成功回调和失败回调暂存
    } else {
      this.successCallBack.push(successCallBack)
      this.failCallBack.push(failCallBack)
    }
  }
}

module.exports = MyPromise
```



#### then方法的链式调用

​	Promise 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数是`rejected`状态的回调函数，它们都是可选的。

`then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

// 判断Promise对象then方法返回值并进行处理
const resolvePromise = (nextValue, resolve, reject) => {
  // 如果在then方法中返回了当前Promise对象，则进行了循环引用，需要错误处理
  if (nextValue instanceof MyPromise) {
    // 返回值是Promise对象，调用该Promise对象的then方法
    nextValue.then(resolve, reject)
  } else {
    resolve(nextValue)
  }
}

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = []
  // 失败的回调函数
  failCallBack = []
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
    // 调用then方法中的回调函数
    while (this.successCallBack.length) {
      this.successCallBack.shift()(this.value)
    }
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
    // 调用then方法中的回调函数
    while (this.failCallBack.length) {
      this.failCallBack.shift()(this.reason)
    }
  }
  // then
  then(successCallBack, failCallBack) {
    // 如果then方法没有传递参数，则使用一个函数作为默认参数
    successCallBack = successCallBack ? successCallBack : value => value
    failCallBack = failCallBack ? failCallBack : reason => { throw reason }
    // then方法返回一个Promise对象
    let promise = new MyPromise((resolve, reject) => {
      // 当前状态是fulfilled，执行成功回调
      if (this.status === FULFILLED) {
        // 传递给下一个Promise对象的值是then方法的返回值
        let nextValue = successCallBack(this.value)
        resolvePromise(nextValue, resolve, reject)
        // 当前状态是rejected，执行失败回调
      } else if (this.status === REJECTED) {
        let nextValue = failCallBack(this.reason)
        resolvePromise(nextValue, resolve, reject)
        // 当前状态是padding, 如异步函数调用。暂时将成功和失败的回调存储。
      } else {
        this.successCallBack.push(() => {
          let nextValue = successCallBack(this.value)
          resolvePromise(nextValue, resolve, reject)
        })
        this.failCallBack.push(() => {
          let nextValue = failCallBack(this.reason)
          resolvePromise(nextValue, resolve, reject)
        })
      }
    })
    return promise
  }
}

module.exports = MyPromise
```



#### 捕获错误

​	前面的版本我们没有进行错误处理，我们接下来在可能出现错误的地方，进行错误处理，让代码更严谨一点

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

// 判断Promise对象then方法返回值并进行处理
const resolvePromise = (promise, nextValue, resolve, reject) => {
  // 如果在then方法中返回了当前Promise对象，则进行了循环引用，需要错误处理
  if (promise === nextValue) return reject(new TypeError(`Chaining cycle detected for promise #<Promise>`))
  // 如果在then方法中返回了当前Promise对象，则进行了循环引用，需要错误处理
  if (nextValue instanceof MyPromise) {
    // 返回值是Promise对象，调用该Promise对象的then方法
    nextValue.then(resolve, reject)
  } else {
    resolve(nextValue)
  }
}

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = []
  // 失败的回调函数
  failCallBack = []
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
    // 调用then方法中的回调函数
    while (this.successCallBack.length) {
      this.successCallBack.shift()(this.value)
    }
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
    // 调用then方法中的回调函数
    while (this.failCallBack.length) {
      this.failCallBack.shift()(this.reason)
    }
  }
  // then
  then(successCallBack, failCallBack) {
    // 如果then方法没有传递参数，则使用一个函数作为默认参数
    successCallBack = successCallBack ? successCallBack : value => value
    failCallBack = failCallBack ? failCallBack : reason => { throw reason }
    // then方法返回一个Promise对象
    let promise = new MyPromise((resolve, reject) => {
      // 当前状态是fulfilled，执行成功回调
      if (this.status === FULFILLED) {
        // 为了能拿到Mypromise实例对象，需要异步执行函数
        setTimeout(() => {
          try {
            // 传递给下一个Promise对象的值是then方法的返回值
            let nextValue = successCallBack(this.value)
            resolvePromise(promise, nextValue, resolve, reject)
          } catch (e) {
            reject(e)
          }
        }, 0)
        // 当前状态是rejected，执行失败回调
      } else if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let nextValue = failCallBack(this.reason)
            resolvePromise(promise, nextValue, resolve, reject)
            // 当前状态是padding, 如异步函数调用。暂时将成功和失败的回调存储。
          } catch (e) {
            reject(e)
          }
        }, 0);
      } else {
        this.successCallBack.push(() => {
          setTimeout(() => {
            try {
              let nextValue = successCallBack(this.value)
              resolvePromise(promise, nextValue, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0)
        })
        this.failCallBack.push(() => {
          setTimeout(() => {
            try {
              let nextValue = failCallBack(this.reason)
              resolvePromise(promise, nextValue, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0);
        })
      }
    })
    return promise
  }
}

module.exports = MyPromise
```

#### Promise.resolve

```js
  // Promise.resolve 静态方法
  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }
```



#### Promise.all

`Promise.all()`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```js
const p = Promise.all([p1, p2, p3]);
```

上面代码中，`Promise.all()`方法接受一个数组作为参数，`p1`、`p2`、`p3`都是 Promise 实例，如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。另外，`Promise.all()`方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。

`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

（1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

```ts
  // Promise.all 静态方法
  static all(array) {
    let result = [];
    let index = 0;
    // Promise.all 返回一个Promise对象
    return new MyPromise((resolve, reject) => {
      for (let i = 0; i < array.length; ++i) {
        MyPromise.resolve(array[i]).then(value => {
          result[i] = value
          ++index
          // 当all参数数组中所有项执行结束后执行resolve方法
          if (index === array.length) {
            resolve(result);
          }
        }, err => reject(err))
      }
    })
  }
```



#### 最终版Promise

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

// 判断Promise对象then方法返回值并进行处理
const resolvePromise = (promise, nextValue, resolve, reject) => {
  // 如果在then方法中返回了当前Promise对象，则进行了循环引用，需要错误处理
  if (promise === nextValue) return reject(new TypeError(`Chaining cycle detected for promise #<Promise>`))
  // 如果在then方法中返回了当前Promise对象，则进行了循环引用，需要错误处理
  if (nextValue instanceof MyPromise) {
    // 返回值是Promise对象，调用该Promise对象的then方法
    nextValue.then(resolve, reject)
  } else {
    resolve(nextValue)
  }
}

class MyPromise {
  constructor(executor) {
    executor(this.resolve, this.reject)
  }
  // Promise状态
  status = PENDING
  // 成功的的值
  value = undefined
  // 失败的原因
  reason = undefined
  // 成功的回调函数
  successCallBack = []
  // 失败的回调函数
  failCallBack = []
  // pending -> fulfilled
  resolve = value => {
    if (this.status !== PENDING) return
    this.status = FULFILLED
    this.value = value
    // 调用then方法中的回调函数
    while (this.successCallBack.length) {
      this.successCallBack.shift()(this.value)
    }
  }
  // pending -> rejected
  reject = reason => {
    if (this.status !== PENDING) return
    this.status = REJECTED
    this.reason = reason
    // 调用then方法中的回调函数
    while (this.failCallBack.length) {
      this.failCallBack.shift()(this.reason)
    }
  }
  // then
  then(successCallBack, failCallBack) {
    // 如果then方法没有传递参数，则使用一个函数作为默认参数
    successCallBack = successCallBack ? successCallBack : value => value
    failCallBack = failCallBack ? failCallBack : reason => { throw reason }
    // then方法返回一个Promise对象
    let promise = new MyPromise((resolve, reject) => {
      // 当前状态是fulfilled，执行成功回调
      if (this.status === FULFILLED) {
        // 为了能拿到Mypromise实例对象，需要异步执行函数
        setTimeout(() => {
          try {
            // 传递给下一个Promise对象的值是then方法的返回值
            let nextValue = successCallBack(this.value)
            resolvePromise(promise, nextValue, resolve, reject)
          } catch (e) {
            reject(e)
          }
        }, 0)
        // 当前状态是rejected，执行失败回调
      } else if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let nextValue = failCallBack(this.reason)
            resolvePromise(promise, nextValue, resolve, reject)
            // 当前状态是padding, 如异步函数调用。暂时将成功和失败的回调存储。
          } catch (e) {
            reject(e)
          }
        }, 0);
      } else {
        this.successCallBack.push(() => {
          setTimeout(() => {
            try {
              let nextValue = successCallBack(this.value)
              resolvePromise(promise, nextValue, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0)
        })
        this.failCallBack.push(() => {
          setTimeout(() => {
            try {
              let nextValue = failCallBack(this.reason)
              resolvePromise(promise, nextValue, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0);
        })
      }
    })
    return promise
  }

  catch(failCallback) {
    return this.then(undefined, failCallback)
  }

  // Promise.resolve 静态方法
  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }

  // Promise.all 静态方法
  static all(array) {
    let result = [];
    let index = 0;
    // Promise.all 返回一个Promise对象
    return new MyPromise((resolve, reject) => {
      for (let i = 0; i < array.length; ++i) {
        MyPromise.resolve(array[i]).then(value => {
          result[i] = value
          ++index
          // 当all参数数组中所有项执行结束后执行resolve方法
          if (index === array.length) {
            resolve(result);
          }
        }, err => reject(err))
      }
    })
  }
}

module.exports = MyPromise
```

