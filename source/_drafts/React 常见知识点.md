## React 常见知识点

### 元素

描述了屏幕上的内容，是构建React应用的最小砖块，定义一个元素：

```javascript
const element = <h1>Hello World</h1>
```

将元素渲染为DOM

```javascript
// 来一个根元素，将其交给ReactDOM管理
<div id='root'></div>
// 定义一个元素
const element = <h1>Hello World</h1>
// 将元素渲染为DOM
ReactDOM.render(element, document.getElementById('root'));
```

> **元素一旦被创建后不可更改，如果要更新的话，只能创建一个全新的元素，传入`ReactDOM.render()`**
>
> **React只会更新需要更新的部分，不会更新整个DOM，比如UI树中的文本标签内容发生了变化，只会更新文本标签**

### 组件和props

**组件**类似于JavaScript函数，接受入参(props)，返回描述展示内容的React元素，类似于只读属性。

定义一个函数组件:

```javascript
function Welcome(props) {
  return (
  	<h1>Hello, {props.name}</h1>
  )
}
```

使用一个函数组件：

```javascript
const el = <Welcome name='Joel'/>;
// 将{name: Joel} 作为props传入组件
ReactDOM.render(el, document.getElementById('root'));
```

>  **组件的命名建议使用首字母大写的驼峰命名法，用于区分HTML的标签**
>
>  **props由外界组件传入，是只读的属性，通过this.props获取**

### State和声明周期

State类似于组件内部的私用变量，可以被组件随意更改。

在Class组件中声明state：

```javascript
class Clock extends React.Component {
  constructor(props) {
    // 调用父类构造函数
    super(props);
    // 声明当前自己的state，date就是组件内部的state
    this.state = {date:new Date()};
  } 
  render() {
    return (
    	<div>
      	<h1>Hello World</h1>
      	<!---组件的state被传入了h2标签中--->
      	<h2>Now: {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}
```

组件生命周期，组件被挂载 -> 组件被卸载

```javascript
class Clock {
  componentDidMount() {
    // 组件被渲染到DOM中，组件被挂载
    // 可以随意向class中添加不参与数据流的额外字段
    this.timeID = setInterval(()=>{},1000)
  }
  componentWillUnmount() {
    // 当前组件将要被卸载
  }
}
```

> **只能在constructor初始化state，只能通过setState()来更改**
>
> **setState(updater, [callback])不会立即更新**
>
> **多个setState()会被合并,如果要使用之前的值，传入一个函数(prevStat, props) => {}**
>
> **state可以向下传递，父组件或兄弟组件是无法获取其他组件的state**

