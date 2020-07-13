---
Tags: [ReactNative, tips]
---



### React Native Tips

#### 添加全局变量，React Native是单页面应用

```javascript
global.varName = 1;
// 在任意模块输出 console.log(varName) 都为1
```

#### 指定模拟器运行

```shell
react-native run-ios --simulator "iPhone 11 Pro"
```

#### 函数组件props

创建的自定义组件，在使用时，默认会将所有的字段包裹在`props = {key:val}`中传递到组件内部

```javascript
const MyComp = (props) => {
  return (
    // 使用时，需要调用props.key
 		<Text>{props.title}</Text>
  )
}
<MyComp title={'你好'}></MyComp>

```

