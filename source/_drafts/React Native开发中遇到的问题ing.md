### 如果8081端口已经被其他程序占用了怎么办

- 干掉占用端口的进程

终端输入以下命令，会显示当前占用8081端口的进程

```shell
sudo lsof -i :8081
```

随后kill掉相关进程

```shell
kill -9 <PID>
```

- 或者换个端口继续

```shell
react-native start --port=8090
```

###  Keystore file '../android/app/debug.keystore' not found for signing config 'debug'.

在android目录下，执行

```shell
keytool -genkey -v -keystore debug.keystore
  -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000
```

### 使用pod install 出现 `xcrun: error: SDK iphoneos cannot be located`

首先确定一下原因

```shell
xcrun -k --sdk iphoneos --show-sdk-path
// xcrun: error: SDK "iphoneos" cannot be located
// xcrun: error: SDK "iphoneos" cannot be located
// xcrun: error: unable to lookup item 'Path' in SDK 'iphoneos'
```

如果是以上输出，在命令行，使用 `xcode-select` 切换到当前Xcode应用程序

```shell
// 选中当前的Xcode程序
sudo xcode-select --switch /Applications/Xcode.app
```

### 使用Hook封装的组件，在外面改变子组件的状态，子组件状态没有发生变化

封装了一个alert组件，如下

```javascript
import React, {useState} from 'react';
// 使用Modal包裹弹窗内容
import {Modal, View, Text, TouchableOpacity } from 'react-native';
let CustomAlert = (props) => {
  const [alertVisible, setAlertVisible] = useState();
  
  return (
  	<Modal 
    	visible={alertVisible}
			transparent={true}
    >
    	<View></View>
    </Modal>
  )
}

// 外界调用
import CustomAlert from './CustomAlert';

let parent = () => {
  
  const [show, setShow] = useState();
  function hanldeClick() {
    setShow(true);
  }
  
  return (
  	<Button onPress={hanldeClick}></Button>
		<CustomAlert alertVisible={show} />
  )
}
```

结果，点击了Button后，虽然parent中 `show` 变为 `true` ，但是弹窗并没有显示。一定是 `Alert` 的`alertVisible`依旧是`false`。

- 使用`useEffect`，在组件 `didUpdate` 中将props属性值赋值给 `CustomAlert` 组件

在`CustomAlert`中添加

```javascript
// useEffect(func, [arg])，如果arg不传，类似于 componentDidMount 和 componentDidUpdate
useEffect(() => {
  setAlertVisible(props.alertVisible);
});
```

- 使用ref，将Alert的`alertVisible`暴露给外界，直接在外界调用即可，否则组件组件内部将组件隐藏，但是外界的show，Alert无法修改

```javascript
import {useImperativeHandle, forwardRef} from 'react'
let DTAlert = (props, ref) => {

    const [alertVisible, setAlertVisible] = useState(false);
    useImperativeHandle(ref, () => ({
        // 暴露给外界的接口
        changVisible: (v) => {
            setAlertVisible(v);
        }
    }));
}
export defautlt DTAlert = forwardRef(DTAlert);

// 外界调用
import {useRef} from 'react';
const Parent = () => {
  const alertRef = useRef();
  function hanldeClick() {
    alertRef.current.changVisible(true);
  }
  
  return (
  	<CustomAlert ref={alertRef} />
  )
  
}
```

#### yarn 安装依赖时 `An unexpected error occurred: https://raw.githubusercontent.com/xxx, connect ECONNREFUSED 151.101.228.133:443".`

网络问题，开启VPN，改一下DNS：`114.114.114.114` 或者`8.8.8.8`