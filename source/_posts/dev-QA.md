---
layout: post
title: iOS开发过程中遇到的问题及解决方案
date: 2018-05-15
categories: iOS开发
tags: iOS 小技巧 经验之谈
---

记录一下自己在日常的iOS开发过程中遇到的一些问题以及解决方法，方便以后查阅。

### SVN无法上传.a文件

Xcode自带的svn和Versions以及一些其它工具都有默认忽略`.a`文件。

在Versions中手动添加文件：
选择Versions的菜单View--\>Show Ignored Items，这样就会显示出ignored的文件，找到你要上传的`.a`文件，右键`Add`就可以了。


### Xcode unable to run app in simulator

Xcode 界面，`shift + command +2` 快速进入`devices`窗口，查看当前的可用设备，找到无法启动的模拟器，删除并重新创建一个新的模拟器，再次运行该模拟器即可


### Cannot assign to self out of a method in the init family?

初始化方法必须以init开头，并遵循驼峰命名法则

### file has been modified since the precompiled header was built

在Mac终端删除derivedData：
```shell
rm -rf ~/Library/Developer/Xcode/DerivedData 
```
重新clean build


### 使用Storyboard页面跳转时，tableView的ContentOffset 莫名偏移

不要勾选xib文件的`Adjust Scroll View Insets`

### UITableView setEditing:YES animation:YES 没有动画效果

因为后面再次调用了reloadData导致的。解决方法如下：

- 方案一：

```swift
[CATransaction begin];
[CATransaction setCompletionBlock: ^{

  //animation 结束之后要运行的代码 

}]; 
[_tblView setEditing:YES animated:YES]; 
[CATransaction commit];
```

- 方案二：

```swift
[UIView animateWithDuration:0.3f
                 animations:^{
                    [self.tableView setEditing:YES animated:NO];
                 }
                 completion:^(BOOL finished){
                   // animation 结束之后要运行的代码
}];
```





### The document "MainStoryboard.storyboard" could not be opened. The operation couldn’t be completed. (com.apple.InterfaceBuilder error -1.) Check the console log for additional information.

1.open storyboard as `source code` 

2.find `interredMetricsTieBreakers` block

3.remove all `<segue reference = “” />`

4.open storyboard as `interFace Builder`


### The document “ xxxx.xib" could not be opened.

解决方案：

1.删除xib的 source code 里的冲突

2.右键.xib文件 -\> show file in inspector 文件type：default interface builder cocoa touch xib  (Xcode 7 点击一次ok)

### AVC的View自动加载AView的nib文件

实例：iOS 10.3.3真机
工程中以前有一个名为RewardView的View源文件以及相应的nib文件，再新建一个RewardViewController的源文件不带nib文件，当RewardViewController初始化时，竟然把RewardView的nib文件加载进来啦啦啦，我也是无言以对，难道VC的View加载机制是优先加载命名类似的nib文件？？？

### 用cocoapods安装的第三方库默认iOS Deployment Target 4.3，导致工程编译错误

```swift
post_install do |installer| 
    installer.pods_project.targets.each do |target| 
        target.build_configurations.each do |config| 
            config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '8.0' 
        end 
    end 
end
```
将上面的代码中的`8.0`替换成你要的系统版本号后，粘贴在`podfile`文件后面，执行`pod update` 或者 `pod install` 即可。

#### iOS11tableView cell 分割线问题

没有通过nib加载的cell，都会出现系统的分割线，通过nib加载的cell不会出现分割线

### pods的第三方库有好多Warning
```swift
pod 'SSZipArchive', :inhibit_warnings => true
```

### 自定义tabBarController的tabBar的title显示问题

自定义tabBar的title跟系统的title重影，不要直接设置`vc.title=@“”`，要设置`vc.navigationItem.title=@“”`

### App想用http怎么办

新版xcode已经默认项目使用HTTPS，想要用回http，只需在`info.plist` 添加key：`NSAppTransportSecurity` value:`Dictionary`，在`dictionary`下面添加`NSAllowsArbitraryLoads=YES`的键值对即可

### Xcode 8默认打印了一堆无用的log，怎么办？

在Xcode选项中依次选择 Product -\> Scheme -\> Edit Scheme，调出工程Scheme编辑面板；或者使用快捷键 `Shift+Command+,` 调出Scheme编辑面板。

选中Run选项，在`Environment Variables`里面添加 `OS_ACTIVITY_MODE=Disable`，即可告别多余的日志信息。

但是真机调试的话，NSLog日志并不会出现，那我们就只用宏定义代替：
```swift
#define CCLog(...) printf("%f %s\n",[[NSDate date] timeIntervalSince1970],[[NSString stringWithFormat:__VA_ARGS__] UTF8String]);
```

### tableView Cell分割线位置

```swift
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath{
    
    // Remove separator inset
    if ([cell respondsToSelector:@selector(setSeparatorInset:)]) {
        [cell setSeparatorInset:UIEdgeInsetsMake(0, tableView.frame.size.width, 0, 0)];
    }
    
    // Prevent the cell from inheriting the Table View's margin settings
    if ([cell respondsToSelector:@selector(setPreservesSuperviewLayoutMargins:)]) {
        [cell setPreservesSuperviewLayoutMargins:NO];
    }
    
    // Explictly set your cell's layout margins
    if ([cell respondsToSelector:@selector(setLayoutMargins:)]) {
        [cell setLayoutMargins:UIEdgeInsetsMake(0, tableView.frame.size.width, 0, 0)];
    }
}
```


### 如何实现斜纹背景

项目中需要实现这样的效果：
![][image-1]

每个半小时区间需要斜纹背景填充，可以用UIView或者CALayer实现。
UIView实现：
```swift
 - (void)drawRect:(CGRect)rect {
    
    // 获取当前context
    CGFloat width = self.frame.size.width;
    CGFloat height = self.frame.size.height;
    CGContextRef ctx = UIGraphicsGetCurrentContext();

    // 画矩形，填充背景色
    CGContextAddRect(ctx, CGRectMake(0, 0, width, height));
    [[UIColor colorWithHex:@"#e9e9e9"] set];
    CGContextFillPath(ctx);
    
    // 画斜纹线条
    CGFloat offset = 50;
    [[UIColor colorWithHex:@"#dadada"] set];
    NSInteger count = ceilf((width + offset) * 1.0f / 5.0);
    
    NSMutableArray *startPoints = [NSMutableArray array];
    for (int i = 0; i < count; i ++){
        CGPoint startPoint = CGPointMake(5 * i - offset, height);
        NSValue *value = [NSValue valueWithCGPoint:startPoint];
        [startPoints addObject:value];
    }
    NSMutableArray *endPoints = [NSMutableArray array];
    for (int i = 0; i < count; i ++){
        CGPoint endPoint = CGPointMake(5 * i, 0);
        NSValue *value = [NSValue valueWithCGPoint:endPoint];
        [endPoints addObject:value];
    }
    
    for (int i = 0; i < startPoints.count; i ++){
        
        CGPoint startPoint = [[startPoints objectAtIndex:i] CGPointValue];
        CGPoint endPoint = [[endPoints objectAtIndex:i] CGPointValue];
        CGContextMoveToPoint(ctx, startPoint.x, startPoint.y);
        CGContextAddLineToPoint(ctx, endPoint.x, endPoint.y);
        CGContextStrokePath(ctx);
        
    }
    
}
```

CALayer实现：
```swift
- (CAShapeLayer *)invalidLayerWidthFrame:(CGRect)frame{
    
    // 创建CGPath
    CGMutablePathRef path = CGPathCreateMutable();
    // 创建layer，设置layer绘制颜色
    CAShapeLayer *oneLayer = [CAShapeLayer layer];
    oneLayer.fillColor = [UIColor whiteColor].CGColor;
    oneLayer.strokeColor = [UIColor blueColor].CGColor;
    oneLayer.lineWidth = 1.0f;
    oneLayer.masksToBounds = YES;
    oneLayer.frame = frame;
    oneLayer.backgroundColor = [UIColor lightGrayColor].CGColor;
    
    // 定义斜线间距、起始点与终止点X轴偏移量
    CGFloat offsetX = 50;
    CGFloat spaceX = 5;
    NSInteger count = ceilf((frame.size.width + offsetX) * 1.0f / spaceX);
    
    NSMutableArray *startPoints = [NSMutableArray array];
    for (int i = 0; i < count; i ++){
        CGPoint startPoint = CGPointMake(spaceX * i - offsetX, frame.size.height);
        NSValue *value = [NSValue valueWithCGPoint:startPoint];
        [startPoints addObject:value];
    }
    NSMutableArray *endPoints = [NSMutableArray array];
    for (int i = 0; i < count; i ++){
        CGPoint endPoint = CGPointMake(spaceX * i, 0);
        NSValue *value = [NSValue valueWithCGPoint:endPoint];
        [endPoints addObject:value];
    }
    
    for (int i = 0; i < startPoints.count; i ++){
        
        CGPoint startPoint = [[startPoints objectAtIndex:i] CGPointValue];
        CGPoint endPoint = [[endPoints objectAtIndex:i] CGPointValue];
        CGPathMoveToPoint(path, NULL, startPoint.x, startPoint.y);
        CGPathAddLineToPoint(path, NULL, endPoint.x, endPoint.y);
    }
    
    oneLayer.path = path;
    return oneLayer;
}
```

### Xcode启动图正确添加后不显示

先利用Assets方式添加启动图：

1、在Assets.xcassets -\> LaunchImage图片集中添加各种分辨率启动图片

2、如有`LaunchScreen.storyboard`文件，关闭Xcode General选项卡中 `use as Launch Screen`选项

3、`General`选项改动，`Launch Images Source` 改为 `LaunchImage图片集`；`Launch Screen File` 置空；
改动后重启一下Xcode才能选择已有的启动图片集，然后删除已安装的App再运行才能看到效果😂


### 蓝牙不能自动扫描

创建好`CBCentralManager` 后需要在`viewController` 的 `viewDidAppear`时执行扫描

```swift
mCentralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];

- (void)viewDidAppear:(BOOL)animated{
    
    [super viewDidAppear:animated];
    CBUUID *serviceUUID = [CBUUID UUIDWithString:@"00001101-0000-0100-8000-00805F9B34FB"];
    NSArray *servicesArr = [NSArray arrayWithObjects:serviceUUID,nil];
    [mCentralManager scanForPeripheralsWithServices:servicesArr options:@{CBCentralManagerScanOptionAllowDuplicatesKey:@NO}];
}
```

### pod update 获取不到最新repo

先删除本地的repos，再setup，终端执行以下命令：
```swift
sudo rm -rf ~/.cocoapods/repos/master
pod setup
```

### iPhone X导航栏多了一条线
![][image-2]

看了一下导航栏的view层级，没有发现什么高度为1的view，然后突然醒悟到，可能是导航栏背景图片尺寸的问题，换了高度为88的图片就没事了。


### LaunchScreen加载不到图片

将需要加载的图片，转为JPEG格式的图片，或者tiff格式，具体原因不明，如果出现启动无法加载，删除APP重装，或者重启手机也可以解决

### 网络图片含有中文字符，导致sdwebimgage无法加载

需要将URL使用utf8编码格式一下

```swift
// iOS 9 之前
imgUrl = \[urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
// iOS 9 之后
imgUrl = \[urlStr stringByAddingPercentEncodingWithAllowedCharacters:\[NSCharacterSet URLQueryAllowedCharacterSet]];
```
### SVN version is locked
以CornerStone 为例，workingCopy，右键 -> Clean
### SVN 报错 "An error occurred while contacting the repository.”
删除钥匙串中SVN相关密码，重启SVN客户端。
### ScrollView代理scrollViewDidScroll:和scrollViewDidEndScrollingAnimation:执行了同样的业务代码，怎么防止执行多次呢？
```swift
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    // 去执行did end scroll animation，
    [NSObject cancelPreviousPerformRequestsWithTarget:self];
    [self performSelector:@selector(scrollViewDidEndScrollingAnimation:) withObject:scrollView afterDelay:0.1];
}
- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView {
    [NSObject cancelPreviousPerformRequestsWithTarget:self];
    // your code
}
```
### Xcode使用cocoapods打包出来的ipa包里面包含了第三方库，增加了包体积。
删掉podfile里面的use_frame!
### webview激活键盘后，系统收起键盘会导致一系列问题，比如界面卡死，web内容消失等诡异现象
自己主动收起键盘
### TabBar的未选中item颜色被渲染为灰色了
```swift
// 使用原来的渲染
UIImage *originImage = [[UIImage imageWithName:@“”] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
UITabBarItem *barItem = [UITabBarItem alloc] initWithTitle:@“title” image:originImage selectedImage:selectedImage];
```
### xcode-select: error: tool ‘xcodebuild’ requires Xcode

xcodebuild需要调用Xcode去执行命令，但是xcode-select指向的路径是/Library/Developer/CommandLineTools，不是Xcode路径，所以将路径切换到Xcode路径下就OK了。

```shell
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

[image-1]:	/images/stripe.png
[image-2]:	/images/navi_line.png
