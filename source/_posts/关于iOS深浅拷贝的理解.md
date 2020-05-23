---
layout: post
title: 关于iOS深浅拷贝的理解
date: 2017-12-07
categories: iOS开发
tags: [iOS, 深浅拷贝]
---
在iOS开发过程中，我们要对某个对象进行拷贝的时候，一般会用`copy`和`mutableCopy`两种方法，在刚接触iOS的时候，我一般把`copy`称为浅拷贝，`mutableCopy`称为深拷贝，但是随着不断踩坑，才发现对不同类型的对象执行拷贝操作，得到的结果跟自己的预期并不一致。然后看了网上诸多关于深浅拷贝的文章，再结合自己写代码测试，才慢慢理解到，其实iOS中的深浅拷贝得到的结果，是跟拷贝对象的类型有关系的。
<!-- more -->


在iOS开发中，一般情况下，拷贝对象大概分为3种：非集合对象、集合对象、自定义对象。好了，话不多讲，以下就是针对这三种不同类型对象所写的深浅拷贝测试代码。

### 1、iOS中非集合类的深浅拷贝

非集合类对象深浅拷贝示例代码：

```swift
- (void)stringCopy{
	NSString *str = @"123456";
	// 内容内存地址不变，共用同一块内存
	NSString *cpyStr = [str copy];
	// 内容地址改变，另外开辟一块内存空间，内容可变
	NSMutableString *mutCpyStr = [str mutableCopy];

	NSLog(@"NSString内容地址：str = %p; copyStr = %p; mutStr = %p;",str,cpyStr,mutCpyStr);
	// str = 0x10e6d9068; copyStr = 0x10e6d9068; mutStr = 0x604000253f20;

	NSLog(@"NSstring指针地址：str = %p; copyStr = %p; mutStr = %p;",&str,&cpyStr,&mutCpyStr);
	// str = 0x7fff51525be8; copyStr = 0x7fff51525be0; mutStr = 0x7fff51525bd8;

}
- (void)mutableStringCopy{

   NSMutableString *formatStr = [NSMutableString stringWithFormat:@"呵呵哒🙄"];
   // 内容地址改变，开辟一块内存空间，内容不可变
   NSString *cpyStr = [formatStr copy];
   // 内容地址改变，开辟一块内存空间，内容可变
   NSMutableString *mutCpyStr = [formatStr mutableCopy];

   NSLog(@"NSMutableString内容地址：str = %p; copyStr = %p; mutStr = %p;",formatStr,cpyStr,mutCpyStr);
   // str = 0x600000249690; copyStr = 0x600000249cc0; mutStr = 0x600000249750;

   NSLog(@"NSMutableString指针地址：str = %p; copyStr = %p; mutStr = %p;",&formatStr,&cpyStr,&mutCpyStr);
   // str = 0x7fff51525be8; copyStr = 0x7fff51525be0; mutStr = 0x7fff51525bd8;

}
```

由上面的代码可以看出，`非集合对象`的拷贝规律是这样的：

- 不可变的非集合对象`copy`，指针拷贝，共用同一片内存；
- 不可变的非集合对象`mutableCopy`，创建新内存空间，成为可变对象。
- 可变的非集合对象`copy`，创建新内存空间，成为不可变对象；
- 可变的非集合对象`mutableCopy`，创建新内存空间。

### 2、iOS中集合类的深浅拷贝

#### 集合类的拷贝主要分为下面三种类型：

- 浅复制(shallow copy)：在浅复制操作时，对于被复制对象的每一层都是指针复制。
- 深复制(one-level-deep copy)：在深复制操作时，对于被复制对象，至少有一层是深复制。
- 完全复制(real-deep copy)：在完全复制操作时，对于被复制对象的每一层都是对象复制。

来自苹果官方的解释图例：[官方链接](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html "官方链接")
![](https://snowyblog.oss-cn-shenzhen.aliyuncs.com/CopyingCollections.png)

集合的深浅拷贝示例代码：

```swift
- (void)collectionCopy{

	NSString *str = [NSString stringWithFormat:@"呵呵哒"]();
	NSMutableString *mutStr = [NSMutableString stringWithFormat:@"萌萌哒"]();
	NSArray *arr = [NSArray arrayWithObjects:@"one", @"two", @"three", nil]();

	// 不可变集合对象只包含非集合对象
	NSArray *imutArr1 = [NSArray arrayWithObjects:str, mutStr, nil]();
	// 浅复制，创建一个指针直接指向已有内存空间
	NSArray *cpyImutArr1 = [imutArr1 copy]();
	// 深复制，创建新的集合内存空间，新瓶装老酒
	NSMutableArray *mutCpyImutArr1 = [imutArr1 mutableCopy]();
	// 创建新的集合内存空间，并且对集合内元素进行 copy 操作
	NSMutableArray *initArr1 = [\[NSMutableArray alloc]() initWithArray:imutArr1 copyItems:YES];
	// 完全深复制，如果元素是自定义类型，需要实现NSCoding协议
	NSArray *deepCpyArr1 = [NSKeyedUnarchiver unarchiveObjectWithData:\[NSKeyedArchiver archivedDataWithRootObject:imutArr1]()];

	NSLog(@"\n imutArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&imutArr1, imutArr1, imutArr1[0](), imutArr1[1](), imutArr1[2]());
	// imutArr1 info : pointer = 0x7fff5d946bc8; content = 0x604000443960; first = 0x604000037520; second = 0x604000442d30; third = 0x6040004446e0

	NSLog(@"\n cpyImutArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&cpyImutArr1, cpyImutArr1, cpyImutArr1[0](), cpyImutArr1[1](), cpyImutArr1[2]());
	// cpyImutArr1 info : pointer = 0x7fff5d946bc0; content = 0x604000443960; first = 0x604000037520; second = 0x604000442d30; third = 0x6040004446e0

	NSLog(@"\n mutCpyImutArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&mutCpyImutArr1, mutCpyImutArr1, mutCpyImutArr1[0](), mutCpyImutArr1[1](), mutCpyImutArr1[2]());
	// mutCpyImutArr1 info : pointer = 0x7fff5d946bb8; content = 0x604000444290; first = 0x604000037520; second = 0x604000442d30; third = 0x6040004446e0

	NSLog(@"\n initArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&initArr1, initArr1, initArr1[0](), initArr1[1](), initArr1[2]());
	// initArr1 info : pointer = 0x7fff5d946bb0; content = 0x604000442e80; first = 0x604000037520; second = 0x604000231aa0; third = 0x6040004446e0

	NSLog(@"\n deepCpyArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&deepCpyArr1, deepCpyArr1, deepCpyArr1[0](), deepCpyArr1[1](), deepCpyArr1[2]());
	// deepCpyArr1 info : pointer = 0x7fff5d946ba8; content = 0x604000443210; first = 0x604000231ce0; second = 0x604000444710; third = 0x6040004441d0

	// 可变集合对象包含集合对象
	NSMutableArray *mutArr2 = [NSMutableArray arrayWithObjects:str, mutStr, arr, nil]();
	// 深复制，创建新的集合内存空间，内容不变，新的集合不可变
	NSMutableArray *cpyMutArr2 = [mutArr2 copy]();
	// 深复制，创建新的集合内存空间，内容不变
	NSMutableArray *mutCpyMutArr2 = [mutArr2 mutableCopy]();

	NSLog(@"\n mutArr2 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&mutArr2, mutArr2, mutArr2[0](), mutArr2[1](), mutArr2[2]());
	// mutArr2 info : pointer = 0x7fff53507bc8; content = 0x60400025e2a0; first = 0x60400023fa40; second = 0x60400025d2b0; third = 0x60400025f710

	NSLog(@"\n cpyMutArr2 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&cpyMutArr2, cpyMutArr2, cpyMutArr2[0](), cpyMutArr2[1](), cpyMutArr2[2]());
	// cpyMutArr2 info : pointer = 0x7fff53507bc0; content = 0x60400025d760; first = 0x60400023fa40; second = 0x60400025d2b0; third = 0x60400025f710

	NSLog(@"\n mutCpyImutArr1 info : pointer = %p; content = %p; first = %p; second = %p; third = %p",&mutCpyMutArr2, mutCpyMutArr2, mutCpyMutArr2[0](), mutCpyMutArr2[1](), mutCpyMutArr2[2]());
	// mutCpyImutArr1 info : pointer = 0x7fff53507bb8; content = 0x60400025d310; first = 0x60400023fa40; second = 0x60400025d2b0; third = 0x60400025f710
}
```

通过代码和日志可以看出：

- `不可变集合`的拷贝规律大概是这样的：

`[immutableObject copy]` **浅复制，指针复制**

`[immutableObject mutableCopy]` **深复制，不是完全复制，容器是新的可变容器，内容还是旧的**

- `可变集合`的拷贝规律大概是这样的：

`[mutableObject copy]` **深复制 返回对象为immutable**

`[mutableObject mutableCopy]` **深复制，不是完全复制，容器是新的可变容器，内容还是旧的**

若想要实现完全拷贝，则需要使用下面的代码：
```swift
NSArray *newArr = [NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:oldArr]];
```

至于拷贝后的集合是否可变，则跟旧集合的是否可变一致。值得注意的是，如果集合内含有自定义的类对象，则该类需要实现 `NSCoding` 协议。

如果根据集合重新`init`一个新的集合出来，就像这样子：
```swift
NSMutableArray *newArr = [[NSMutableArray alloc] initWithArray:arr copyItems:YES];
```

那么生成的新集合中的元素，并不一定都是完全拷贝的。因为执行上面的代码，只会对集合内的元素执行 `copy`方法。

### 3、iOS中自定义对象的深浅拷贝

iOS中自定义对象要想实现拷贝，必须要实现`NSCopying`或者`NSMutableCopying`协议。
自定义对象实现拷贝示例代码:

```swift
// Cat.h
# import <Foundation/Foundation.h>

@interface Cat : NSObject <NSCopying, NSMutableCopying>

@property (nonatomic, assign) NSUInteger age;
@property (nonatomic, copy) NSString *name;

@end

// Cat.m
# import "Cat.h"

@implementation Cat

- (instancetype)copyWithZone:(NSZone *)zone{

	return self;
}

- (instancetype)mutableCopyWithZone:(NSZone *)zone{

	Cat *cat = [[Cat alloc] init];
	cat.name = @"hello";
	return cat;
}

@end

```

从代码可以看出，自定义类要遵守`NSCopy`协议或者`NSMutableCopy`协议，拷贝协议的代码由自己来实现，那么自定义类对象执行`copy`或者`mutableCopy`所得的结果，无论是指针层面的浅拷贝，还是完全拷贝，都是由自己来决定的。值得注意的是，如果集合里面包含了自定义类对象，那么要用`NSKeyedArchiver/NSKeyedUnarchiver`实现集合的完全拷贝，则需要自定义类实现`NSCoding`协议。

```swift
- (id)initWithCoder:(NSCoder *)aDecoder{

	self = [super init];
	if (self == nil){
	return nil;
	}
	self.name = [aDecoder decodeObjectForKey:@"name"];
	// 依次写下想要decode的属性
	return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder{

	[aCoder encodeObject:self.name forKey:@"name"];
	// 依次写下想要encode的属性
}
```










