---
title: 改变函数this指向的apply()、call()、bind() 方法
date: 2018-10-19 21:03:01
tags: JavaScript
categories: JavaScript
---

_不同函数运行时会存在不同的this指向，总是指向调用函数的对象。有些时候，我们希望通过改变函数的this指向来实现某些功能。常见的如继承等等_

### 函数的this指向
  根据函数的调用方式可以分为以下几种，它们分别有各自的this指向。
![](/img/改变函数this/函数的this指向.png)

#### 1. 使用call()改变函数的this指向
1.1 基本语法
使用`fn.call(object, parm1, parm2....)`会立即调用`fn`, 第一个参数是要给`fn`绑定的对象,也就是要将`fn`的this指向更改为对应的对象，从第二个参数起，是传给`fn`的参数，参数的长度不限，参数之间使用空格分开。
```javascript
var a = 1;
var b = 2;
const obj = {
	a: 3,
	b: 4
}
function fn(a, b) {
	console.log(this.a + this.b);	
	console.log(a + b);
}
fn(5, 6);		       // 此时的this指向的是Window  所以输出的结果是 3、11
fn.call(obj, 7, 8);	   //此时的this指向了obj, 所以输出的结果是 7、15
```
#### 2. 使用apply()改变函数的this指向
2.1 基本语法
`fn.apply(object, arr)`, 和`call`非常类似，也会立即调用`fn`, 也会更改`fn`的this指向为接受到的第一个参数。不同之处在于`apply()`只接受两个参数，第二个参数是一个数组，将一个数组作为参数传递给`fn`。
```javascript
var a = 1;
var b = 2;
const obj = {
	a: 3,
	b: 4
}
function fn(a, b) {
	console.log(this.a + this.b);	
	console.log(a + b);
}
fn(5, 6);		       // 此时的this指向的是Window  所以输出的结果是 3、11
fn.call(obj, [7, 8]);  //此时的this指向了obj, 所以输出的结果是 7、15, 需要将需要的参数按照数组的方式进行传递
```

#### 3. 使用bind() 改变函数的this指向

3.1 基本语法

`fn.bind(object, para1, para2...)`,在形式上和`fn.call()`非常相似，传递的参数是一模一样的，唯一的区别是`fn.bind()方法使用时`,不会立即调用`fn`.上面的另外两个方法`fn.apply()`和`fn.call()`会在使用时立即调用`fn`函数。使用`bind()`返回的是原函数改变this之后产生的新函数。

使用场景

```javscript
// 下面是一个使用场景 
function LateBloomer() {
  this.petalCount = Math.ceil(Math.random() * 12) + 1;
}

// 在 1 秒钟后声明 bloom 回调函数的this指向的是window
// 如果这个地方换成apply或者bind，也能改变this的指向，但是函数会立即执行，
// setTimeout设置的时间间隔会失效
LateBloomer.prototype.bloom = function() {
  window.setTimeout(this.declare.bind(this), 1000);
};

LateBloomer.prototype.declare = function() {
  console.log('I am a beautiful flower with ' + this.petalCount             +'petals!');
};

var flower = new LateBloomer();
flower.bloom();  // 一秒钟后, 调用 'declare' 方法
```

**总结**：存在可能合理，不同的使用场景下，选择不同的方法实现效果。达到目的即可，关于更深层的理解，以后有了深入理解会补充。