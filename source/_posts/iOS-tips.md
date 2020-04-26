---
layout: post
title: iOS开发小技巧
date: 2018-04-18
categories: iOS开发
tags: iOS 小技巧 经验之谈
---


本文持续更新，主要记录自己在iOS开发过程中用到的一些小技巧，助力快速开发

### UIButton动态设置文字出现闪烁

Button type不要使用system，要用custom

### tableView除掉多余空白行：

```swift
tableView.tableFooterView = [UIView new];
```

### 自定义leftBarButtonItem导致左滑返回手势失效

```swift
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithImage:img style:UIBarButtonItemStylePlain target:self action:@selector(onBack:)];
self.navigationController.interactivePopGestureRecognizer.delegate = (id <UIGestureRecognizerDelegate> )self;
```

### ViewController中的scrollView出现莫名的frame位移或者contentOffset有偏差

```swift
// iOS 11 之前
self.automaticallyAdjustScrollViewInsets = NO;
self.edgesForExtendedLayout = UIRectEdgeNone;
// iOS 11
scrollView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
```


### 页面滚动时，隐藏navigationbar

```swift
navigationController.hidesBarsOnSwipe = YES;
```


### 拉伸图片的时候怎么才能让图片不变形

```swift
UIImage *image = [[UIImage imageNamed:@"xxx"] stretchableImageWithLeftCapWidth:10 topCapHeight:10];
```

### 把tableview里cell的小对勾的颜色改成别的颜色

```swift
tableView.tintColor = [UIColor redColor];
```

### Navigationbar弄成透明的而不是带模糊的效果

```swift
[self.navigationController.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];
self.navigationController.navigationBar.shadowImage = [UIImage new];
self.navigationController.navigationBar.translucent = YES;
```

### 改变UITextField placeholder的颜色和位置

```swift
// 继承UITextField 重写
- (void)drawPlaceholderInRect:(CGRect)rect {
    [[UIColor blueColor] setFill];
    [self.placeholder drawInRect:rect withFont:self.font lineBreakMode:UILineBreakModeTailTruncation alignment:self.textAlignment];
}
```

### 自定义NavigationBar的titleView

```swift
UIView *titleView = [UIView alloc] init];
self.navigationItem.titleView = titleView;
// iOS 11 如果titleView上的button不响应，自定义titleView，加入@property (nonatomic, assign) CGSize intrinsicContentSize;
CustomTitleView *titleView = [CustomTitleView alloc] init];
titleView.intrinsicContentSize = CGSizeMake(100,44);
self.navigationItem.titleView = titleView;
```

### UIScorllView ContentSize 通过添加Masonry layout约束确定尺寸

```swift
// 新建scrollView 添加约束确定frame
UIScrollView *scrollView = [[UIScrollView alloc] init]; 
[self.view addSubview:scrollView]; 
[scrollView mas_makeConstraints:^(MASConstraintMaker *make) { 
    make.edges.equalTo(self.view);
}];
// 添加container到scrollView上，添加container到scrollview的各边边距，在通过指定container宽高决定contentSize
UIView *containerView = [[UIView alloc] init];
[scrollView addSubview:containerView];
[containerView mas_makeConstraints:^(MASConstraintMaker *make) {
// 指定container与scrollView边距，很关键
    make.edges.equalTo(scrollView);
// 指定container宽度
    make.width.equalTo(scrollView);
// 指定container高度，+1保证scrollView的bounds
    make.height.equalTo(scrollView).offset(1);
}];
```

### WebView 获取HTML的title

```swift
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation{
  
    [webView evaluateJavaScript:@"document.title" completionHandler:^(id _Nullable obj, NSError * _Nullable error) {

        self.title = [NSString stringWithFormat:@"%@",obj];
    }];
}
```

### 监听UITextField内容变化

```swift
[textField addTarget:self action:@selector(textfiledEditChanged:) forControlEvents:UIControlEventEditingChanged];
```

### Animation改变constraint

```swift
constraint.constant = 100;
[UIView animationWithDuration:.3f animations:^{
  [self.view layoutIfNeeded];
}];
```

### 不通过Xcode打包ipa

Xcode `archive`后，在xcode的`Archives`中找到相应的`Archive包`，显示包内容，在`Products/Applications/`路径下面找到应用程序，在桌面新建`Payload`文件夹，将应用程序拖入`Payload`文件夹，压缩，改压缩包后缀为`.ipa`。

### XCode使用自定义字体

- 1、添加自定义字体.ttf到XCode工程；
- 2、info.plist添加key:`<Fonts provided by application.`，value类型为`Array`，添加自定义字体名到Array；
- 3、确保Build Phases -\> Copy Bundle Resources 里面包含你的字体文件，不然不会显示字体
- 4、代码调用
```swift
UIFont *digitFont = [UIFont fontWithName:@"Impact" size:38];
```

### Xcode 忽略一些废除接口警告

```swift
   //-Wunused-variable 变量未使用 -Wdeprecated-declarations 使用废弃方法 -Wdocumentation 文档注释 
pragma clang diagnostic push
pragma clang diagnostic ignored "-Wdeprecated-declarations"
   //code 废弃的方法
pragma clang diagnostic pop
```

### TableView 点击空白收起键盘

```swift
UITapGestureRecognizer \*tableViewGesture = \[\[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tableViewTouchInSide)];
//是否取消点击处的其他action
tableViewGesture.cancelsTouchesInView = NO;
[self.subTableView addGestureRecognizer:tableViewGesture];

- (void)tableViewTouchInSide{
    // ------结束编辑，隐藏键盘
    [self.view endEditing:YES];
}
```

### Xcode 标记

```swift
// MARK: -
// TODO: -
// FIXME: -
```

### iOS 11 tableView 刷新跳动

```swift
tableView.estimatedRowHeight = 0;
tableview.estimatedSectionHeaderHeight = 0;
tableview.estimatedSectionFooterHeight = 0;
```

### Xcode 添加预编译文件
1、新建`.pch`文件，勾选target
2、`building setting`选项卡中，搜索`pch`，找到 `Prefix Header`，添加 `$(SRCROOT)/YourProjectName/YourPCHFileName`
### iOS状态栏样式设置
- 通过代码设置样式或者隐藏，必须要在`Info.plist`中添加字段`View controller-based status bar appearance : NO`
// 1、通过在工程的Info.plist文件中加入字段改变全局样式
`UIStatusBarStyle : UIStatusBarStyleDefault/UIStatusBarStyleLightContent`
// 2、代码全局设置样式，首先在`Info.plist`中加入字段，然后在`APPDelegate`中写下代码
`View controller-based status bar appearance : NO`
```Swift
[UIApplication sharedApplication].statusBarStyle = UIStatusBarStyleLightContent;
```
- 单独设置状态栏背景颜色
```Swift
- (void)setStatusBarBackgroundColor:(UIColor *)color { 
    UIView *statusBar = [[[UIApplication sharedApplication] valueForKey:@"statusBarWindow"] valueForKey:@"statusBar"]; 
    if ([statusBar respondsToSelector:@selector(setBackgroundColor:)]){
        statusBar.backgroundColor = color; 
        } 
}
```
### 图片不变形拉伸
```Swift
// 低于iOS 5.0
- (UIImage *)stretchableImageWithLeftCapWidth:(NSInteger)leftCapWidth topCapHeight:(NSInteger)topCapHeight
// 高于iOS 5.0
- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets
```
### 优雅地退出App
```Swift
// 暴力强退，不会调用applicationWillTerminate，会被拒绝
exit(0);
// 苹果建议使用，会调用applicationWillTerminate，会被拒绝
abort(), assert(condition);
// 前两种退出会给用户造成一种App崩溃的赶脚，下面来一个优雅的方式，会使用到私有接口
// suspend 挂起程序，会给用户一种退入后台的感觉，然后延时调用terminateWithSuccess，直接干死App，或者使用上面的暴力手段，或者执行一个不存在的方法
[[UIApplication sharedApplication] performSelector:@selector(suspend) withObject:nil afterDelay:0];
[[UIApplication sharedApplication] performSelector:@selector(terminateWithSuccess) withObject:nil afterDelay:0.4];
```
### NavigationBarItem间距调整
```Swift
// iOS 11,需要自定义button，重写AlignmentRectInsets属性
UIButton *customButton = [UIButton buttonWithType:UIButtonTypeCustom];
// you should do this in your own custom class
customButton.overrideAlignmentRectInsets = UIEdgeInsetsMake(0, x, 0, -x); 
customButton.translatesAutoresizingMaskIntoConstraints = NO;
UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithCustomView:customButton]
// fixed space item 调整间距
UIBarButtonItem *positiveSeparator = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace target:nil action:nil];
positiveSeparator.width = 8;
self.navigationItem.leftBarButtonItems = @{positiveSeparator, item, ...}
```
### XCode日志老是输出The connection to service named com.apple.commcenter.coretelephony.xpc was invalidated
```Shell
// 终端运行下面的命令
xcrun simctl spawn booted log config --mode "level:off"  --subsystem com.apple.CoreTelephony
```
### iOS中获取WIFI的SSID(名字)，非私有方法
```Swift
// 导入头文件
#import <SystemConfiguration/CaptiveNetwork.h>
// 获取ssid
- (NSDictionary *)fetchSSIDInfo {
    NSArray *interfaceNames = CFBridgingRelease(CFCopySupportedInterfaces());
    NSDictionary *SSIDInfo;
    for (NSString *interfaceName in interfaceNames){
        SSIDInfo = CFBridgingRelease(CNCopyCurrentNetworkInfo((__bridge CFStringRef)interfaceName));
        BOOL isNotEmpty = (SSIDInfo.count > 0);
        if (isNotEmpty){
            break;
        }
    }
    return SSIDInfo;
}
// iOS12 需要加入WIFI权限
```

