---
title: es6 new 
tags: [JavaScript, ES6]
---



#### let 和 const

**let变量的作用域**

let用来声明新的变量，由let声明的变量，只在声明的代码块内有效。在代码块外、未声明变量前，调用该变量都会报错 `ReferenceError`。

```javascript
{
  // 在声明前引用
  console.log('inner let a is ',a); // ReferenceError
  let a = 1;
  var b = 3;
}
	// 在代码块外面
  console.log('out print let a is ', a); // ReferenceError
	console.log('out print var b is ', b); // 3
```

**for循环作用域**

使用let，在循环变量声明中，每次循环都会生成一个新的变量，JavaScript引擎会记住上次循环值。

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

**暂时性死区**

在代码块内，如果有let声明的变量，那么在声明前，对变量进行调用都会出现ReferenceError，且不受外界影响，这个区域就是暂时性死区。

```javascript
// 内部的x有着自己的作用域，不受外界x影响。
let x = 5;
{
  // zone begin
  let x = y;
  console.log(y);
  let y; // zone end
  y = 3;
}
```



var声明的变量，在整个文件内有效，而且可以变量提升，即在未声明前就可以使用，并且为`undefined`

```javascript
console.log(a); // undefined 调用时变量a并没有声明，但是已经可以访问
var a = 3;
```





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

```javascript
// 调用改函数时，会直接返回一个Promise
async getAmount(p1, p2) {
  let price1 = await getProductPrice(p1);
  let price2 = await getProductPrice(p2);
  // 所有的await执行完毕，才会return
  return price1 + price2;
}
// 类似的函数写法
function getAmount(p1, p2) {
 	return new Promise((resolve, reject) => {
    
  }) 
}
// 调用
getAmount(p1, p2).then(res => {
  console.log('total price = ', res);
});
```

`async` 函数内部的 `return` 返回值会成为`then` 回调函数的参数，在函数内部抛出错误，会导致`Promise` 对象变为`reject`状态，被`catch`回调函数接收。

```javascript
async fetchRemote() {
  let response = await fetch('url');
  // 触发Promise的reject
  if (response.code !== 200) throw new Error(response.msg);
  // 触发Promise的resolve
  return response.data;
}
```

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





