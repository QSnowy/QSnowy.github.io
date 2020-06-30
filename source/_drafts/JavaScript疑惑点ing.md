function.apply()

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

函数还可以添加属性值，感觉很厉害的样子



arguments： 函数内部所有参数集合，并非是Array，箭头函数里不能用

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

