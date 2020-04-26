---
layout: post
title: SDWebImage代码解析
date: 2019-03-07
categories: iOS
tags: [iOS, SDWebImage, 源码阅读]
---

SDWebImage是GitHub上面一个很火的iOS和MacOS平台下，图片下载、加载、管理开发第三方库。


整个源码大概分为：图片管理、下载器、下载任务、缓存、编码解码、UI展示几个模块。
- 图片管理：SDWebImageManager
- 下载器：SDWebImageDownloader
- 下载任务：SDWebImageDownloaderOperation
- 缓存：SDWebImageCache
- 解码：SDWebImageIOCoder、SDWebImageGIFCoder
- UI展示：UIImageView+WebCache 等
- 配置其他：SDImageCacheConfig等
<!-- more -->


**先说一下，整体的图片下载流程：**
给定一个URL给`Manager`后，会先用`Cache`检测是否有对应的缓存图片，如果没有缓存且需要下载，`Manager`持有的`Downloader`会生成一个`URLSession`，将`URL`转化为一个`Request`，然后把`Request`和`URLSession`包装在`Operation`中，将`Operation`加入到自己的`下载队列`中，开始执行下载任务；任务完成或者异常，经过`系统的Request代理`回调，拿到图片数据或异常，通过`Coder`将数据解压为图片，利用`Cache`缓存在内存和磁盘中，通过`block回调`给到主线程进行处理。

### 图片管理

采用单例模式，初始化后默认会持有`SDWebImageCache`和`SDWebImageDownloader`两个实例，便于进行图片缓存和下载管理。不过用户也可以通过下面的方法自行指定

```swift
- (nonnull instancetype)initWithCache:(nonnull SDImageCache *)cache downloader:(nonnull SDWebImageDownloader *)downloader
```
同时提供代理方法，为用户提供URL过滤功能，需要自己实现代理方法。

#### 常用方法：
使用Mananger下载指定URL图片，监听下载进度，完成回调，返回一个可取消的下载任务。

```swift
- (nullable id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url
                                              options:(SDWebImageOptions)options
                                             progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                            completed:(nullable SDInternalCompletionBlock)completedBlock;

```

### 下载器

采用单例模式，用户可以指定URLSessionConfiguration，所有URL请求都会使用这一个session配置。
#### 常用方法：
指定session配置
```swift
- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration
```
开始一个URL下载请求，监听进度，完成回调，返回一个下载Token，用户可以使用这个Token取消下载。一个URL会产生一个Operation，加入下载队列，进行网络请求。
```swift
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock
```

取消一个下载任务
```swift
- (void)cancel:(nullable SDWebImageDownloadToken *)token
```

### 下载任务
这里面定义了一个任务可交互的协议，`SDWebImageDownloaderOperationInterface`，协议规定了任务的初始化方法、添加任务进度和完成的回调的方法、任务的取消方法以及证书的添加。任务继承与NSOperation，一个请求对应了一个任务，这些任务会放在下载管理单例中的一个下载队列中，可以按先入先出的顺序执行，也可以按堆栈的后入先出的顺序执行。

任务在收到请求回应后，会将接受到的图片数据解码成为UIImage对象，再通过Block进行回调给主线程。

#### 常用方法：
初始化一个任务，这是一个实现的协议方法。
```swift
- (nonnull instancetype)initWithRequest:(nullable NSURLRequest *)request
                              inSession:(nullable NSURLSession *)session
                                options:(SDWebImageDownloaderOptions)options
```
添加任务进度回调
```swift
- (nullable id)addHandlersForProgress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                            completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock
```

### 缓存
通过这个模块，将下载完成的图片，存储到内存或者磁盘中。同样是单例模式，用户可以指定缓存路径。使用了`SDMemoryCache:NSCache`进行内存存储，内部使用了`NSMapTable<KeyType, ObjectType> *weakCache`这个属性进行实际的内存存储工作。
#### 常用方法：
指定缓存路径
```swift
- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns
                       diskCacheDirectory:(nonnull NSString *)directory
```
存储图片到内存或者磁盘，图片通过编码工具转为二进制数据，最终存储在磁盘中。
```swift
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock
```
从内存或者磁盘中找到与key对应的图片
```swift
- (nullable UIImage *)imageFromCacheForKey:(nullable NSString *)key;
```

清除内存和磁盘缓存
```swift
- (void)clearMemory;
- (void)clearDiskOnCompletion:(nullable SDWebImageNoParamsBlock)completion;
```

### 编码解码
框架中有IOCoder和GIFCoder，我这里只展示IOCoder，下载完的图片会通过转化为NSData，同时从网络获取的数据也会被解码转化为image。
#### 关键方法：
将图片根据不同的图片格式，转化为二进制数据。
```swift
- (NSData *)encodedDataWithImage:(UIImage *)image format:(SDImageFormat)format{
    // 判断图片格式，最后会保存为PNG或者JPEG格式，这里省略了一些源码
    NSMutableData *imageData = [NSMutableData data];
    CFStringRef imageUTType = [NSData sd_UTTypeFromSDImageFormat:format];
    // 创建 image destination.
    CGImageDestinationRef imageDestination = CGImageDestinationCreateWithData((__bridge CFMutableDataRef)imageData, imageUTType, 1, NULL);
    // 设置图片方向信息
    NSMutableDictionary *properties = [NSMutableDictionary dictionary];
    NSInteger exifOrientation = [SDWebImageCoderHelper exifOrientationFromImageOrientation:image.imageOrientation];
    [properties setValue:@(exifOrientation) forKey:(__bridge NSString *)kCGImagePropertyOrientation];
    // 添加图片 to the destination.
    CGImageDestinationAddImage(imageDestination, image.CGImage, (__bridge CFDictionaryRef)properties);
    // 完成，将图片写入数据.
    CGImageDestinationFinalize(imageDestination);    
    return [imageData copy];
} 
```
将图片解码，只是简单调用了系统接口。
```swift
- (UIImage *)decodedImageWithData:(NSData *)data{
       if (!data) {
        return nil;
    }
    UIImage *image = [[UIImage alloc] initWithData:data];
    image.sd_imageFormat = [NSData sd_imageFormatForImageData:data];
    return image;
}
```
渐进式解析图片，主要用到了`CGImageSourceCreateIncremental`函数处理。

```swift
- (UIImage *)incrementallyDecodedImageWithData:(NSData *)data finished:(BOOL)finished {
    if (!_imageSource) {
        // 创建渐进式ImageSource
        _imageSource = CGImageSourceCreateIncremental(NULL);
    }
    UIImage *image;
    // 将累积的数据传进来，不一定是完整的
    CGImageSourceUpdateData(_imageSource, (__bridge CFDataRef)data, finished);
    if (_width + _height == 0) {
        // 获取source信息，从信息中拿到图片的宽、高、方向信息，省略了代码 
        CFDictionaryRef properties = CGImageSourceCopyPropertiesAtIndex(_imageSource, 0, NULL);
    }
    if (_width + _height > 0) {
        // 拿到了有效信息，开始创建图片
        CGImageRef partialImageRef = CGImageSourceCreateImageAtIndex(_imageSource, 0, NULL);
        if (partialImageRef) {
            image = [[UIImage alloc] initWithCGImage:partialImageRef scale:1 orientation:_orientation];
        }
    }
    // 如果传入了结束标志，说明当前图片的数据下载完了，返回生成好的image
    if (finished) {
        if (_imageSource) {
            CFRelease(_imageSource);
            _imageSource = NULL;
        }
    }
    return image;
}

```
解压缩图片，主要用`CGBitmapContextCreate`创建位图。
```swift
// 该函数接收参数有data、width、height、bitsPerComponent、bytesPerRow、space、bitmapInfo
// data 如果不为NULL，那么它应该指向一块大小至少为 bytesPerRow * height 字节的内存；如果为NULL，那么系统就会为我们自动分配和释放所需的内存，所以一般指定NULL即可；
// width height 位图要输出的宽度和高度，宽高方面有多少颗pixel；
// bitsPerComponent 一个像素中每个独立的颜色分量使用的bit数，一个像素由RGB三色组合，这个参数代表R、G、B每个分量使用几个bit；
// bytesPerRow 每一行使用的字节数，设为0，系统会自动优化，自行设定大小至少为 width * bytes per pixel字节，bytesPerPixel是每颗像素需要多长字节去存储；
// space 颜色空间，常用的有<RGB、CMYK、BGR、HSB>，不同的空间对相同的数值有不同的解析方式，最终得到的图片色彩也不一样；
// bitmapInfo alpha信息表示alpha在像素中排在哪个位置、颜色分量是否浮点、像素格式的字节顺序；iOS使用小端字节序；
CG_EXTERN CGContextRef __nullable CGBitmapContextCreate(void * __nullable data,
    size_t width, size_t height, size_t bitsPerComponent, size_t bytesPerRow,
    CGColorSpaceRef cg_nullable space, uint32_t bitmapInfo)
    CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

- (nullable UIImage *)sd_decompressedImageWithImage:(nullable UIImage *)image {
    @autoreleasepool{
        
        CGImageRef imageRef = image.CGImage;
        // 当前设备的色彩空间，实现用了once_token
        CGColorSpaceRef colorspaceRef = SDCGColorSpaceGetDeviceRGB();
        BOOL hasAlpha = SDCGImageRefContainsAlpha(imageRef);
        // 位图信息：32位 + alpha信息
        CGBitmapInfo bitmapInfo = kCGBitmapByteOrder32Host;
        bitmapInfo |= hasAlpha ? kCGImageAlphaPremultipliedFirst : kCGImageAlphaNoneSkipFirst;
        size_t width = CGImageGetWidth(imageRef);
        size_t height = CGImageGetHeight(imageRef);
        // 传入各种参数
        CGContextRef context = CGBitmapContextCreate(NULL,
                                                     width,
                                                     height,
                                                     kBitsPerComponent,
                                                     0,
                                                     colorspaceRef,
                                                     bitmapInfo);
        if (context == NULL) {
            return image;
        }
        // 在上下文绘制图片
        CGContextDrawImage(context, CGRectMake(0, 0, width, height), imageRef);
        // 创建图片
        CGImageRef imageRefWithoutAlpha = CGBitmapContextCreateImage(context);
        UIImage *imageWithoutAlpha = [[UIImage alloc] initWithCGImage:imageRefWithoutAlpha scale:image.scale orientation:image.imageOrientation];
        CGContextRelease(context);
        CGImageRelease(imageRefWithoutAlpha);
        return imageWithoutAlpha;
    }
}
```

### UI展示
为UIImageView和UIButton提供了类别方法，可以方便的设置相应的图片属性。
#### 常用方法：
为UIImageView设置图片
```swift
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder;
```
添加图片加载Activity，定义在`UIView+WebCache`中。
```swift
- (void)sd_setShowActivityIndicatorView:(BOOL)show;
```

自己做的SDWebImage框架脑图，赶脚有点大😂

图片管理模块

![](/images/sdwebmanger.png)

图片下载工具模块

![](/images/sddownloader.png)

图片缓存模块

![](/images/sdcache.png)

图片解码模块

![](/images/sdcoder.png)

扩展工具模块

![](/images/sdextension.png)

其他模块

![](/images/sdother.png)


