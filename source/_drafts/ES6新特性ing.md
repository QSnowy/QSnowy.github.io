---
title: es6 new 
tags: [JavaScript, ES6]
---



#### let 和 const

let声明的变量，只在声明的代码块内有效。在未声明变量前，使用该变量会报错ReferenceError。

在循环变量声明中，每次循环都会生成一个新的变量，JavaScript引擎会记住上次循环值。

```javascript
let a = [];
for (let i = 0; i < 10; i ++) {
  a[i] = function() {
    console.log(i);
  }
}
a[6]();
// 6
// a数组中的每个函数都会保留一个变量i
```

for循环声明区是父级作用域，循环内部是子级作用域。

```javascript
for (let a = 0; a < 10; a ++) {
  let a = 'abc';
  console.log(a);
  // abc
}
```

var声明的变量，在整个文件内有效。



#### 变量的解构赋值

看个例子

```javascript
let user = {id: 'xYo38SZj'};
let {id: userId} = user;
console.log('userId',userId);
// log: userId: xYo38SZj
```

`user` 的 `id` 值被赋值给了 `userId`

#### 字符串模板



#### async函数

`async` 函数返回一个 Promise 对象，可以使用`then`方法添加回调函数。当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。



#### 变量提升

一般情况下，我们是先声明变量再使用，但是有了提升的概念后，我们可以先使用，再声明变量。实际上变量和函数的声明位置并未改变，而是在代码编译阶段被放入了上下文中的最前面。

```javascript
num = 8;
console.log(`num is ${num}`); 	// num is 8
var num;
```

变量提前，但初始化并未提前，依旧需要自己初始化变量。

```javascript
console.log(`num is ${num}`); // num is undefined
num = 8;
var num;
```





