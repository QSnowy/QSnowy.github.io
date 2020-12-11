---
title: JavaScript
---

### 基本数据类型

- javascript中数据类型：`number`、`boolean`、`string`、`null`、`undefined`、`object`
- Javascript的对象：狭义的对象（object）、Array、Function
- 自动判断为false的值：`undefined`、`null`、0、NaN、false、""
- 由于`null`转为`number`时为0，为了避免出错，定义了`undefined`类型

### 对象

- 普通的对象就是{key:value}对组成的数据结构，key一定是字符串，也叫属性；value可以是任意值；

- 指向的对象的变量，只是引用值；指向基本数据的变量，是拷贝值

  ```javascript
  var a = {name: 'boy'};
  var b = a;
  a.name = 'girl';
  a = 1;
  // 
  console.log(`a is ${a} b is ${b}`); // a = 1 b = {name: 'girl'}
  var x = 5;
  var y = x;
  x = 1;
  console.log(`x is ${x} y is ${y}`) // x = 1 y =5;
  
  ```

- 遍历对象的可遍历属性，for ... in object

  ```javascript
  var o = {name: 'bob', age: 19, weight: 50}
  for (var k in o) {
    // k = name; k = age; k = weight;
    if (o.hasOwnProperty(k)) {
      	// 判断当前属性是否是自身，而非继承过来的
    }
  }
  ```

- with可以批量操作对象属性，但是由于不经意间创建全局变量，不建议使用

  ```javascript
  let obj = {name: 'xue'};
  with (obj) {
    name = 'quan';
    // 由于obj没有age属性，会自动创建一个全局变量age
    age = 18;
  } 
  ```


### 函数

在JavaScript中函数与其他数据类型一样，被看作对象，可以进行传递和返回。函数的声明，会被变量提升到首行。

```javascript
foo();
function foo() {
  console.log('function is hoist');
}
```

- var声明的变量只有在函数作用域内才算做局部变量，其他地方都属于全局变量
- 函数默认返回值是 `undefined`
- 函数默认属性：length会返回参数的个数
- 函数执行的作用域是定义时的作用域，而不是调用时的作用域；

```javascript
var a = 9;
function foo() {
  // 函数作用域中的变量a=9
  console.log(a);
}

function zoo() {
  var a = 2;
  // 此时定义的变量不会影响到函数内部引用的变量
  foo();
}
// 9
```

- 函数参数传递：值传递、地址传递（对象、数组、函数）

```javascript
// 值传递
var a = 9;
function change(val) {
  val = 8;
}
console.log('a is',a)
// 9 函数内部修改的是val指向的9，而不是变量a
// 地址传递
var b = {name: 'xue'};
function foo(v) {
  v = {name: 'quan'}；
  v.name 'zhang';
}
console.log('b is ', b);
// b is {name: 'xue'}; 形参v指向了b的地址，但是内部又将v指向了新的object，所以之前的对象没有发生变化
```

- arguments： 函数内部所有参数集合，只有在函数内部才能使用，并非是Array，箭头函数里不能用，带有一个`callee`属性，返回原函数

```javascript
function add() {
  let num = 0,
      len = arguments.length;
  if (var i = 0; i < len; i ++) {
    num += 1;
  }
  console.log('func has arg count = ', num);
}
add();						// 0
add(1,2,3);				// 3
add(1,2,3,4,5,6);	// 6
```

- 闭包(closure)

为了获取函数内部的变量才有了闭包，本质上，闭包是将函数内部和函数外部连接起来的桥梁。最大用处：可以读取函数内部变量，让变量始终保持在内存中。

```javascript
function IncreateFoo(start) {
 	return function () {
    return start++;
  }
}

// 外界使用，闭包内部保留的传入的数值
var inc = IncreateFoo(5);
inc(); 	// 5
inc();	// 6
inc();	// 7
```



JavaScript的函数上下文，存在`定义时上下文`, `运行时上下文`, `上下文可改变`。

```javascript
function fruit() {};
fruit.proptype = {
  color: 'red',
  say: function() {
    console.log(`my color is $(this.color)`);
  }
}
let f = new fruit();
f.say();
// my color is red
```

函数还可以添加属性值，感觉很厉害的样子。

### 数组

在JavaScript中数组，本质上也是object。

```javascript
let arr = [0, 'abc'];
typeof(arr); // object
```

默认的键值就是常用的索引值：0、1 ...，除了默认的键值，还能够添加自定义的key。

```javascript
let arr = ['a', 'b', 'c'];
arr['name'] = 'array';
```

可以通过直接赋值length属性清空数组

```javascript
let arr = [1,2,3,4];
arr.length = 0; // 清空数组
```

### 错误处理

JavaScript一旦发生错误，就会抛出异常，停止运行。原生`Error`对象都有一个`message`属性，其他可能还有`name` 和 `stack` 信息。

```javascript
var err = new Error('错误信息');
err.meessage; // 错误信息
```

`throw` 语句，手动终止，并抛出错误，或者其他任意类型值

```javascript
function trowAny() {
  throw new Error('stop by user');
  // 其他任意类型
  // throw 1; throw 'error'; throw {name: 'obj'}
}
```

`try...catch...finally` 语句，无论是否发生错误都会执行 `finally` 语句，并且会覆盖 `catch` 中的返回值

```javascript
function foo() {
  try {
		console.log(0);
  	throw 'bug';
	} catch (err) {
  	console.log(1);
  	return true;		// 会延迟在finally后面执行
  	console.log(2); // 不会执行
	} finally {
  	console.log(3);
  	return true;		// 覆盖之前的true，返回值变为false
  	console.log(4); // 不会执行
	}
  console.log(5);		// 不会执行
}
let res = foo();
// 0 1 3
res // false
```



