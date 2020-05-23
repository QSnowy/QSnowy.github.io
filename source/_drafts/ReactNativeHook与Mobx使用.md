---
title: React Native 函数组件如何使用Mobx
tags: [React Native, Hook, Mobx]
---

### Mobx与函数组件的配合使用

之前的Mobx的是配合着类组件来使用的，但是React 在16.8版本后引入了函数组件和Hook，网上的关于函数组件如何使用Mobx管理数据的中文教程又太少。只能自己配合[Mobx迁移文档](https://mobx-react.js.org/recipes-migration)来慢慢摸索。

##### 前置环境：mobx-react 6.x & React 16.8

定义数据model，设定可监听属性。

```javascript
// user.js
import { observable } from 'mobx';
class User {
  // 定义属性
  @observable
  name = '';
	tel = '';
}
```

使用React的 `useContext` 创建全局Store Hook

```javascript
// store.js
// 创建context
const AppContext = React.createContext({
  'userStore': new User()
})
// export 全局 store hook
export useStores = () => {
  return React.useContext(AppContext)
}
```

在组件中使用model

```javascript
// component.js
import { observer } from 'mobx-react';
const comp = () => {
  // 引用store
  const {userStore} = useStores();
  return (
  	<h1>Welcome {userStore.name} thanks visited</h1>
  )
}
// 组价绑定store，触发数据更新监听
export default observer(comp);
```

