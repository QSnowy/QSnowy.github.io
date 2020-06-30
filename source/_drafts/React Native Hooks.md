### Hook的使用

React 16.8新特性，在不使用Class的情况下使用state和其他React特性 

#### useState

```react
import React, { useState } from 'react';
const comp = () => {
  const [count, setCount] = useState(0);
  return (
    <h1>you clicked {count} times</h1>
    <button onClick={() => setCount(count+1)}>Click me</button>
  )
}
```

每次点击按钮， `count` 都会+1，组件的内容也会随之更新。其实 `<h1>you clicked {count} times</h1>` 并没有将 `count` 与标签绑定，而是每次 `setCount()` 时，React都会重新把 `count` 值带入组件内部进行一次渲染。

每次渲染，类似一次组件的副本，保存着渲染时的 `state`，组件内部的函数也是如此。

#### useEffect

组件副作用，可以替代类组件的几个声明周期函数

```javascript
const App = () => {
  // useEffect(func, [参数])
  // 如果[参数]不传递，每次组件刷新都会执行func，相当于didMount和didUpdate声明周期融合
  useEffect(() => {
    console.log('component mounted or updated');
    return () => {
      console.log('destory something');
    }
  })
  // 只有参数变量发生变化时，才会触发函数执行
  useEffect(() => {
    console.log('name is changed', name); 
  }, [name])
  // 传入空参数，组件卸载时，执行函数体
  useEffect(() => {
    console.log('component is unmount');  
  },[])
}
```

### useRef

使用 `useRef`，并配合 `forwardRef` 可以将函数组件内的节点或者接口暴露给外界。

看个例子，将整个组件暴露出去：

```javascript
// TextInput
import React, { useState, useRef, forwardRef, useImperativeHandle } from 'react';
import { Modal, Text, TextInput, TouchableOpacity } from 'react-native';

const MyInput = forwardRef((props, ref) => {
  return (
  	<TextInput ref={ref}>
    </TextInput>
  )
})
// 外界使用
import MyInput from 'MyInput';
const App = () => {
  const inputRef = useRef();
  const press = () => {
    // 激活输入框
    inputRef.current.focus();
  }
  return (
  	<MyInput ref={inputRef}></MyInput>
		<Button onPress={()=>press()}></Button>
  )
}
```

将组件的某个接口暴露出去：

```javascript
const Alert = forwardRef((props, ref) => {
			
	const [vis, setVis] = useState(false);
  const [title, setTitle] = useState(props.title);
	const [content, setContent] = useState(props.content);

  const alertRef = useRef();
  // useImperativeHandle(ref, createHandle, [deps])
  // ref：要暴露的ref<alertRef>，createHandle：暴露的接口或数据
  useImperativeHandle(ref, ()=>({
    show: (title, content) => show(title, content),
    hide: () => hide()
  }))
  
  const show = (title, content) => {
    setVis(true);
  }
  const hide = () => {
    setVis(false);
  }
  
  return (
  <Modal ref={alertRef} visible={vis}>
    	<TouchableOpacity onPress={()=>setVis(false)}>
          <Text>{title}</Text>
    			<Text>{content}</Text>
    	</TouchableOpacity>
    </Modal>
  )
})
// 外界使用
import Alert from 'Alert';
const App = () => {
  const alertRef = useRef();
  const press = () => {
    // 弹窗
    alertRef.current.show('title', 'content');
  }
  return (
  	<Alert ref={alertRef}></Alert>
		<Button onPress={()=>press()}></Button>
  )
}
```









