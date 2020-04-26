---
layout: post
title: iOS HealthKit 接入
date: 2018-08-15
categories: iOS开发
tags: [iOS, HealthKit]
---

HealthKit是苹果提供的一个健康数据接入库，开发人员可以由此读取、写入健康数据，不过依旧是需要先获取相应的接入权限才能使用。
<!-- more -->


以读取和写入健康步数为例，首先用Xcode新建一个空白项目，然后配置项目相应的选项：

1. info.plist 添加`NSHealthUpdateUsageDescription`和`NSHealthShareUsageDescription`;

2. Capabilities选项中打开`HealthKit`;

3. BuildPhases选项中添加`HealthKit.frmaework`;

写一个工具类`HKHandle`，用来判断设备是否支持HealthKit、请求相应权限之类的功能，二话不说上代码。

### HKHandle.h 文件，定义几个常用的接口：

```swift
// HKHandle.h
#import <Foundation/Foundation.h>
#import <HealthKit/HealthKit.h>

@interface HKHandle : NSObject

// HealthKit 实例，全局最好只有一个store
+ (HKHealthStore *)singleStore;
// 默认要访问的HealthKit权限
+ (NSSet *)defaultHKSet;
// 请求获取HealthKit权限
+ (void)requsetDefaultAuth:(void(^)(BOOL suc, NSString *msg))complention;
// 写入数据的设备信息，非必需
+ (HKDevice *)defaultDevice;
// 读取健康数据
+ (void)readDataWithSampleType:(HKSampleType *)type completion:(void(^)(NSArray *results, NSError *error))completion;
// 保存健康数据
+ (void)saveHKObject:(HKObject *)obj completion:(void (^) (BOOL suc, NSError *error))completion;

@end

```

### HKHandle.m文件，接口的具体实现：

生成单例store对象，用于存储健康数据：

```swift
+ (HKHealthStore *)singleStore{
    
    static HKHealthStore *store;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        store = [[HKHealthStore alloc] init];
    });
    return store;
}
```

健康数据类型，这里只有步数类型：

```swift
+ (NSSet *)defaultHKSet
{
    // 步数
    NSSet *set = [NSSet setWithObjects:
                  [HKQuantityType quantityTypeForIdentifier:HKQuantityTypeIdentifierStepCount],nil];
    
    return set;
}
```

判断设备是否支持，请求HealthKit权限：

```swift

+ (void)requsetDefaultAuth:(void(^)(BOOL suc, NSString *msg))complention{
    
    HKHealthStore *store = [self singleStore];
    if (![HKHealthStore isHealthDataAvailable]){
        // 设备不支持 HealthKit
        if (complention){
            complention(NO, @"设备不支持HealthKit");
        }
        return;
    }
    
    // 请求相应健康权限
    NSSet *hkset = [self defaultHKSet];
    [store requestAuthorizationToShareTypes:hkset readTypes:hkset completion:^(BOOL success, NSError * _Nullable error) {
        
        NSLog(@"auth success = %@, error = %@",@(success), error);
        
        if (complention){
            complention(success, error.localizedDescription);
        }
        
    }];
    
}
```
*⚠️健康数据的读取和写入，都需要先获得相应的权限*

读取步数：
HealthKit读取数据，需要一个query对象，步数读取需要`HKSampleQuery`类型的query。

```swift
+ (void)readDataWithSampleType:(HKSampleType *)type completion:(void(^)(NSArray *results, NSError *error))completion{
    
    NSPredicate *predicate = [self todayPredicate];
    
    HKSampleQuery *query = [[HKSampleQuery alloc] initWithSampleType:type predicate:predicate limit:HKObjectQueryNoLimit sortDescriptors:nil resultsHandler:^(HKSampleQuery * _Nonnull query, NSArray<__kindof HKSample *> * _Nullable results, NSError * _Nullable error) {
        
        if (completion){
            
            completion(results, error);
        }
    }];
    
    HKHealthStore *store = [self singleStore];
    [store executeQuery:query];
}

+ (NSPredicate *)todayPredicate{
    
    NSCalendar *calendar = [NSCalendar currentCalendar];
    NSDate *now = [NSDate date];
    NSDateComponents *components = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay fromDate:now];
    NSDate *startDate = [calendar dateFromComponents:components];
    NSDate *endDate = [calendar dateByAddingUnit:NSCalendarUnitDay value:1 toDate:startDate options:0];
    NSPredicate *predicate = [HKQuery predicateForSamplesWithStartDate:startDate endDate:endDate options:HKQueryOptionStrictStartDate];

    return predicate;
}

```

获取到步数的处理：
```swift
- (IBAction)readSteps:(id)sender {
    
    HKSampleType *stepType = [HKSampleType quantityTypeForIdentifier:HKQuantityTypeIdentifierStepCount];
    [HKHandle readDataWithSampleType:stepType completion:^(NSArray *results, NSError *error) {
        // 切换主线程
        dispatch_async(dispatch_get_main_queue(), ^{
           
            if (!results){
                NSLog(@"查询失败了 error = %@",error);
                return ;
            }
            NSInteger sum = 0;
            for (HKQuantitySample *sample in results) {
                double count = [sample.quantity doubleValueForUnit:[HKUnit countUnit]];
                sum += count;
                NSLog(@"count = %@",@(count));

            }
            self.title = [NSString stringWithFormat:@"%@",@(sum)];
        });
    }];
}
```

通用保存健康数据接口：

```swift
// 通用保存接口
+ (void)saveHKObject:(HKObject *)obj completion:(void (^) (BOOL suc, NSError *error))completion{
    
    if (![obj isKindOfClass:[HKObject class]]){
        return;
    }
    HKHealthStore *store = [self singleStore];
    // 保存健康数据
    [store saveObject:obj withCompletion:^(BOOL success, NSError * _Nullable error) {
        
        if (completion){
            
            completion(success, error);
        }
        
    }]; 
}
```

生成步数，调用写入接口:

```swift
// 生成步数，调用接口写入
- (IBAction)writeSetp:(id)sender {
    [self.view endEditing:YES];

    if (_stepCount == 0 || _startTime == nil || _endTime == nil){
        [UIAlertController showSimpleAlertWithTitle:@"提示" message:@"请补充所有选项" presentdViewController:self];
        return;
    }
    
    // 步数
    HKQuantity *steps = [HKQuantity quantityWithUnit:[HKUnit countUnit] doubleValue:_stepCount];
    
    HKQuantitySample *stepSample = [HKQuantitySample quantitySampleWithType:[HKQuantityType quantityTypeForIdentifier:HKQuantityTypeIdentifierStepCount] quantity:steps startDate:_startTime endDate:_endTime device:[HKHandle defaultDevice] metadata:nil];
    
    [HKHandle saveHKObject:stepSample completion:^(BOOL suc, NSError *error) {
        
        if (suc){
            
            [UIAlertController showSimpleAlertWithTitle:@"提示" message:@"写入成功" presentdViewController:self];
            
        }else{
            
            NSLog(@"写入失败，error = %@",error);
        }  
    }]; 
}

```


