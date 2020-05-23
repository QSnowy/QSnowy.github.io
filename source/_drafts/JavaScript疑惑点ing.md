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