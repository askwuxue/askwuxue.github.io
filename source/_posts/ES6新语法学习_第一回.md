---
title: ES6新语法学习_第一回
date: 2018-04-20 23:03:40
tags: ES6
categories: ES6
---

_随着学习的深入，真的发现`ECMAScript5`留下了蛮多坑的，还好，标准不断的更新，语言也会越来越好，简单看一下ES6新的语法，慢慢的更新吧。_

##### 作用域的变化

**let**
_1. 引入块级作用域，let产生产生暂时性死区，取消变量提升。_
阮一峰老师的教程中举了一个`for`循环的中用`var`定义的循环变量导致泄漏为全局变量的问题。

```javascript
var arr = [];
var(var i = 0; i < 10; i++) {
	arr[i] = function() {
	console.log(i);
	}
}
console.log(arr[5]); // 10
```
  我记得以前我大致遇到过同样的问题，但是并不知道是为什么，后来看了阮一峰老师的ES6，瞬间明白。`var`声明的循环变量，其实是一个全局变量。在循环体内部，输出`i`的值是没有问题的。但是除了循环体，这时候全局变量`i`的值已经是10了。其实是一个变量泄露为全局变量的问题。
  使用`let`声明的循环变量，是属于块级作用域的。不会泄露为全局变量，所以使用`let`声明，最后输出的结果是6。

_2. 没有预解析，必须先声明使用，同一个作用域不能重复定义。_

**`const`常量** 
  `const`声明一个常量，实际不是一个变量的值不变，而是指向那个变量的内存地址不变。其实主要是约束引用类型的。
  对于对象来讲，`const`约束了对象的内存地址不可变，也就是一个变量被声明，不能被重新赋值，这里的不能被重新赋值指的是，你不能对该变量的内存地址进行修改。如下
```javascript
const obj = {};
	obj = {};
```
  当一个变量被使用`const`声明，该变量的内存地址不可变，接下来，用另一个对象进行赋值（本质是把另一个对象的地址给`obj`变量）是不允许的。
对象是存在堆中的，如果直接对象本身修改是可以的。如下
```javascript
obj.a = 'xxx';
```
  如果想要一个对象真的不可修改，可以使用`Objec.freeze()`这个方法，对象将会被冻结，关于该对象的一切都不可修改。包括该对象的原型。语法如下,`return`: 被冻结的对象。
```javascript
Object.freeze(对象)
```


##### 解构赋值
_结构赋值的本质是模式匹配，只要等号两边的模式相同，左边的变量就会被赋予对应的值。没有被赋值到的值默认是`undefined`_
```javascript
// 1. 基本语法
let [a, b, c] = [1, 2, 3];
// 2. 对象结构赋值
let json = {"name": "wuxue", "age": 20}
let {name:n, age:a} = json;
// 应用：交换两数，接受后台数据
```
##### 字符串的查找
```javascript
// str是否包含某字符串 返回的是boolean
1. str.includes(substr)
// 返回的是索引的位置
2. str.indexOf(substr)
// 判断是否是某字符串开头/结束。返回值为Boolean
3. str.startsWith('substr')/str.endsWith('substr')
// 重复谋和某个字符串，n 重复的次数，返回重复后的string
4. str.repeat(n);
// 在字符串的头部填充字符串
// 如果规定的字符串的长度超出了，填充的字符串会被自动截取
5. str.padStart(targetLength, padString)
6. str.padEnd(padString, padString);
```
##### ES6新增的数组方法
```javascript
// 第一个参数是回调函数  第二个参数是this指向
// 回调函数参数分别为 value(当前值)、index(当前索引)、arr(指向当前调用的arr)
// 箭头函数的话，无法改变this指向

// 加强的for循环
1. arr.forEach()
	callback(value,index,arr);
// 正常情况需要配合return 如果没有return 功能相当于foreach 返回值是一个数组，	将元素组项变换
// 注意：只要是用map 一定要有return 语句

2. arr.map()
    callback(item,index,arr);

//  如果filter 的回调函数 是true的项留下
3. arr.filter()
    callback(item,index,arr) {
        return condition;
    }；

// 某些项满足条件
4. arr.some()

// 每一项满足条件
5. arr.every()
    callback(item,index,arr) {
        return condition;
    }
// 默认prev指arr[0] cur为arr[1]
6. arr.reduce()
    callback(prev,cur,index,arr){
        return prev + cur;
    }

// 从arr的右边向左边
7. arr.reduceRight();
```
##### 新增的对象方法
```javascript
简写写法
1. 属性值和属性一样时,可以只写其中一个
// 比较两个值是否相等 可以用来判断两个对象的是否有同一个引用
2. Object.is(a, b);
// Object.is(NaN,NaN)	true
// Object.is(+0,-0)	false
// 用来合并对象/复制	复制数组/合并 后面的对象中的一致的会覆盖前面的
5. Object.assign(target，souse1，souse2...)
```

##### 函数的新变化
1. 可以有默认参数,参数可以解构

2. 函数的参数默认是已经定义的，不能在函数中重复定义

3. 拓展运算符,reset运算符  `...`展开数组  在函数参数中用拓展运算符接受多余参数  

4. 箭头函数`let funName = (params)=> {}`; 如果只有一个`return`语句,大括号省略,` return`省略

5. 箭头函数中的`this`是定义函数的时候所在的对象，不再是运行时所在对象。

6. 箭头函数中没有`arguments`

7. 箭头函数不能当成构造函数

##### 新增的数据结构
**1. Set数据结构**
```javascript
//会将重复的值过滤掉,具有唯一性
let sets = new Set(['a','b','c']);
sets.add('d');		// 添加一个元素
sets.delete('d');	// 删除一个元素
sets.has('d');		// false
sets.size			// 3 查看个数
sets.clear()		// 清除所有

// 返回一个新的迭代器对象，该对象包含Set对象中的按插入顺序排列的所有元素的值
set.value();
// 与values()方法相同，返回一个新的迭代器对象，该对象包含Set对象中的按插入顺序排列的所有元素的值。
for (let item of sets.keys()) {
    console.log(item);
}
// 返回一个新的迭代器对象，该对象包含Set对象中的按插入顺序排列的所有元素的值的[value, value]数组。
// 为了使这个方法和Map对象保持相似， 每个值的键和值相等。
for (let [k,v] of sets.entries())

sets.forEach((value,index) => {})
```