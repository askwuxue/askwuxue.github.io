---
title: JavaScript中各种for循环的使用场景以及性能
date: 2018-11-02 22:43:10
tags: JavaScript
categories: JavaScript
---

### JavaScript中各种for循环的使用场景以及性能
_JavaScript中的`for`循环，`forEach`循环，`for...in`循环，`for...of`循环。你能清楚的区分在哪些场景下应该那种循环方式吗？让我们来一起看一下。今天在掘金看到一篇文章，自己也算是有了一丢丢的收获，应该可以写一写。_

#### 1. 运行速度
请看下面的测试代码，比较了在同样条件下，`forEach`循环，`for...in`循环，`for...of`循环的耗时情况。
**1.1 创造一个有1000000个元素的数组**
```ts
let arr = Array(1000000);
for (let i = arr.length; i--; i) {
    arr[i] = i;
}
```
**1.2 `forEach`循环，`for...in`循环，`for...of`循环的耗时**
```ts
console.time('for');
for (let i = arr.length; i >= 0; i--) {}
console.timeEnd('for');

console.time('forEach');
arr.forEach((v) => v)
console.timeEnd('forEach');

console.time('for-of');
for (let key of arr) {}
console.timeEnd('for-of');

console.time('for-in');
for (let key in arr) {}
console.timeEnd('for-in');

// 运行时间
// for: 4.077ms
// forEach: 26.605ms
// for-of: 53.678ms
// for-in: 533.47ms
```
​    _上述的结果，在不同的环境下。可能会存在不同，但是整体趋势不变，速度最快的是for循环，然后依次是`ForEach`，`for of`，`for in`_。
**注意：请注意上述1.1，使用Array()生成数组后又将数组的内容进行了重新赋值。因为Array（1000000）生成的是具有1000000空位的数组。对于数组内的空位，循环的处理不同。`ForEach`会跳过空位，`for of`依旧会遍历空位。因此，如果数组内全部都是空位，会发现`forEach`速度最快。具体可以参考阮一峰老师的[ES6教程](https://es6.ruanyifeng.com/#docs/array)**

**1.3 倒序的for循环可能会更快一些**

来看两种for循环的不同写法。

**正序**

```ts
console.time('for-asc');
for (let i = 0; i < arr.length; i++) {}
console.timeEnd('for-asc');
```



**倒序**

```ts
console.time('for-desc');
for (let i = arr.length; i >= 0; i--) {}
console.timeEnd('for-desc');
```

​    _这两种方式的差别，是for循环的中`**condition**`的差别，正序的方式，每一次都要获取数组的长度，然后和`i`进行大小比较。倒序的方式只需要获取一次数组的长度。因此，数据量巨大时。倒序的方式可能更快一点。当然，将数组的长度赋值给一个变量。每一次循环只需要获取一次这个变量的值是最优雅的写法。也就不存在正序块还是倒序快的说法了。通过这个小知识点，其实想表达的是，写代码真的有很多可以推敲的地方。优化代码的地方，这些我们容易形成思维定式的地方。藏着太多的知识点和_



这个地方是倒序，比for循环的正序要快，因为倒序只需要取一次arr.length

#### 2. `for循环`，`for in循环`，`for of循环`，`forEach循环`的差异与使用场景

​    for循环的可读性较差，应该是指循环数组和对象的时候，会有一定的不优雅，产生了其他的几种for循环的变形。这些变形，除了上述，我们测试的效率不同之外，场景和方法也略有不同。

**2.1 `ForEach`**

```ts
// 接受一个回调函数，可以对数组的每一项进行处理。
arr.forEach((item, index, arr) => {})
```

**`forEach`不能像for循环一样使用brake循环跳出当前循环。目前的唯一方法是抛出异常跳出循环，这应该不是我们想要的。**

**2.2 `for...in`**

```ts
for(let key in target) {
    // 做些什么
}
```

`for in` 以任意顺序遍历一个对象的除了Symbol以外的可枚举属性，会遍历原型上的属性。因为，最好不要在数组中使用，一般遍历数组，我们都希望按照索引输出。`for in `会在意料之外

**2.3 ``for of**
```ts
for (variable of iterable) {
    //statements
}
```
for...of语句在可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

**`for of `只在可迭代的对象上才可以执行，普通对象不是一个可迭代对象，所以不可以使用`for of`**



**根据`for循环`，`for in循环`，`for of循环`，`forEach循环`的不同的特性，在不同的场景，选择最合适的方式才是最优解。**