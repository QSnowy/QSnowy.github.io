---
Title: React Native Init
Tags: [ReactNative]
---

##  RN项目初始化

React Native 基于前端语言JavaScript，实现跨平台开发。因此需要安装js全家桶Node.js，使用npm管理工程依赖。

#### 前提条件

- 已经安装Xcode

- 安装Android Studio

- 配置了ADB环境变量

#### MAC环境，使用`homebrew`安装node，watchman，

```shell
brew install node
brew install watchman
```

#### 全局安装`react-native-cli`脚手架，快速创建RN项目

```shell
npm install -g react-native-cli
```

#### 初始化并运行RN项目

```shell
cd <projectDirectory>
react-native init <project>
cd <project>
// 运行ios和android
react-native run-ios
react-native run-android
```

## 项目基本配置

### 1. 导航和底部栏配置

导航栏使用的是官方推荐的[React Navigation](https://reactnavigation.org/docs/getting-started)

#### 导航设置

安装导航依赖

```shell
npm install @react-navigation/native
// 还需要手动安装native的其他依赖
npm install react-native-gesture-handler
// stack 依赖于masked-view, 需要同时安装
npm install @react-navigation/stack
npm install @react-native-community/masked-view
// 也可以将所有依赖放在同一行命令里面
npm install react-native-gesture-handler react-navigation-stack react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

设置App Containers

```javascript
// 管理所有导航堆栈和状态，类似于根控制器
import { NavigationContainer } from '@react-navigation/native'

// 返回一个Object {Screen: s, Navigator: nv }, Navigator包含一堆的screen，只有被包含的screen，才能实现相互跳转
import { createStackNavigator } from '@react-navigation/stack';

// 创建堆栈导航控制器
const Stack = createStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.screen name=""/>
      </Stack.Navigator>
    </NavigationContainer>
  )
}
```

共享ScreenOptions，多个页面公用样式，比如导航栏颜色，标题字体等。

```javascript
function stack() {

 return (
  <Stack.Navigator
    screenOptions={{
      headerStyle: {
        backgroundColor: '#909090',
        // iOS导航栏出现底部横线时，加上
        shadowColor: 'transparent',
      },
      headerTintColor: '#fff',
      headerTitleStyle: {
        fontWeight: 'bold',
      },
    }}>
  </Stack.Navigator> 
 )
}
```

关于Screen，类似于view controller，可以传入options，带入自定义的title

```javascript
<Stack.Screen options={{title: 'customTitle'}}>
```

每一个screen初始化时，默认会传入`navigation`，利用`navigation`就可已跳转到任意在`navigator`中的screen

```javascript
function screen({ navigation }) {
  return (
   <Button onPress={() => navigation.navigate('routeName')} />
  );
}
```

导航页面传参

```javascript
/* page 1 */
function page1({ navigation }) {
  return (
    <Button onPress={() => navigation.push('page2', {id: 10})}/>
  )
}

/* page 2 */
// 从route中获取页面参数
function page2({ route, navigation }) {
  const { id } = route.params
  return (
    <Text>id: {JSON.stringify(id)}</Text>
  )
}
```

#### TabBar设置

安装tab依赖

```shell
npm install @react-navigation/bottom-tabs
```

设置tab navi ctrl

```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

function screen1() {return (<View></View>)}
function screen2() {return (<View></View>)}

// 创建tab navigator
const Tab = createBottomTabNavigator();
export default function App() {
  return (
    <NavigationContainer>          
      <Tab.Navigator>
        <Tab.Screen name=""/>
      </Tab.Navigator>          
    </NavigationContainer>
   )
}
```

[tabBarItem配置](https://reactnavigation.org/docs/bottom-tab-navigator)

```javascript
<Tab.Screen 
  options={{
    // 返回组件，函数，默认参数，可以指定
    tabBarIcon: ({tintcolor}) => (<Image source={imgurl} />)
  }}
/>

<Tab.Navigator 
  tabBarOptions={{
    activeTintColor: '#040404'
  }}
/>
```

安装了新依赖后，记得在`iOS`目录下运行一下 `pod install`

### 2. 页面样式

组件使用类css样式，首先导入`StyleSheet`，接着创建`StyleSheet`实例，最后在相应组件引入样式

```javascript
import { StyleSheet } from 'react-native';
// 定义样式
const styles = StyleSheet.create({ 
  viewStyle: { 
    background: 'red'
  }
})
// 引用样式
<view style={styles.viewStyle} />
```

### 3. 网络请求

使用`fetch`进行网络请求，链式函数调用

```javascript
function login(url, param) {
  // 定义请求头
  let header = {
    // 定义协商好的Content-Type
    'Content-Type': 'application/json',
    'token': 'your auth token'
  }
  // 请求体需要转化
  let body = JSON.stringify(param)
  // 指定请求方法
  let method = 'POST';
  let data = {header: header, method: method, body: body}
  fetch(url, data)
  // 将服务器响应数据转化为json
  .then(responds => responds.json())
  .then(res => {
    // handle json respond
  })
  .catch(err => {
    // handle err
  })
}
```

### 4. 数据存储

简单的组件状态管理，使用`hook`管理，`useState`是react提供通用状态管理钩子函数，接受变量初始值(也可以是返回初始值的函数)，会返回长度=2的数组，首位是变量，末尾是设置变量的函数

```javascript
import { useState } from 'react'
function comp() {
  // 定义组件内的变量
  const [count, setCount] = useState('0');
  return (
    <button onClick={()=>{setCount(count+1)}}>点我+1</button>
  )
}
```

使用`MobX`管理全局状态，安装依赖

```shell
npm install mobx mobx-react
```

由于MobX使用了一些修饰符，还要用到`babel`处理编译

```shell
npm i babel-plugin-transform-decorators-legacy babel-preset-react-native-stage-0 --save-dev
npm i --save-dev @babel/cli @babel/plugin-proposal-decorators @babel/preset-env @babel/runtime babel-plugin-transform-class-properties babel-preset-react-app babel-preset-react-native
```

在babel.config.js或者.babelrc中添加一下配置

```json
{
 presets: ['module:metro-react-native-babel-preset',"react-app"],
 plugins: [
  [
   '@babel/plugin-proposal-decorators',
   {
    'legacy': true
   }
  ]
 ]
}
```

### 5. 组件周期监听

使用`React`的`useEffect`可以进行组件、状态监听，从而做出相应的处理逻辑

```javascript
function example() {
  const [name, setName] = useState('');
  // useEffect(func, [参数])
  // 如果[参数]不传递，每次组件刷新都会执行func，相当于didMount和didUpdate声明周期融合
  useEffect(() => {
    console.log('component mounted or updated') 
  })
  // 只有参数变量发生变化时，才会触发函数执行
  useEffect(() => {
    console.log('name =', name) 
  }, [name])
  // 传入空参数，组件卸载时，执行函数体
  useEffect(() => {
    console.log('component is unmount');  
  },[])
}
```

https://subscription.lightyun.site/link/jWYXz1DpjNrZ2xuC?sub=1&extend=1