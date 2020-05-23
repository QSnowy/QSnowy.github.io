---
title: es6 new 
tags: [JavaScript, ES6]
---



let 和 const

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



变量的解构赋值



字符串模板