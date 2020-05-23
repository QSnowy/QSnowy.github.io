### Navigation和Tab

导入Navigation框架

```shell
npm install react-native-navigation
```

> 如果工程没有自动链接，请升级一下react版本

改下RN入口 `index.js` 的代码

```javascript
import App from './App';
import { Navigation } from "react-native-navigation";
// 首先加载loading，根据用户状态，动态改变rootNavigation
import LoginScreen from './src/views/loading';

// 关掉底部警告提示
console.disableYellowBox = true;
// 注册
Navigation.registerComponent('Loading', () => LoginScreen);
// 应用启动，设置root
Navigation.events().registerAppLaunchedListener(() => {
    Navigation.setRoot({
        root: {
            component: {
                name: 'Loading'
            }
        }
    });
});
```

设置BottomTab，属性值需要包裹在`options`中

```javascript
export const goMain = () => Navigation.setRoot({
    root: {
        bottomTabs: {
            children: [
                {
                    stack: {
                        options: {
                          bottomTab: {
                            icon,
                            selectedIcon,
                            text: title,
                            fontSize: 10,
                          },
                          bottomTabs: {
                            hideShadow: true,
                          }
                        },
                        children: [
                            {
                                component: {
                                    name: 'Home'
                                }
                            }
                        ]
                    }
                }
            ]
        }
    }
})
```

