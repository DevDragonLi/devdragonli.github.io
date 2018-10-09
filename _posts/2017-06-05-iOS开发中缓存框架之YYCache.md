---
layout: post
title: iOS开发中缓存框架之YYCache
date: 2017-06-05
categories: blog
tags: [iOSDev]
description: iOS 开发中总会用到各种缓存，YYCache或许是你最好的选择。性能上有优势.API简单

---

# YYCache使用

#### iOS 开发中总会用到各种缓存，YYCache或许是你最好的选择。性能上有优势.API简单
#### YYDiskCache对YYKVStorage一层封装，缓存方式：数据库+文件

- YYCache.h

```
@interface YYCache : NSObject
// 读取当前数据库名称
@property (copy, readonly) NSString *name;

// memoryCache内存缓存，diskCache文件缓存
@property (strong, readonly) YYMemoryCache *memoryCache;
@property (strong, readonly) YYDiskCache *diskCache;

// 可通过下面三种方法来实例化YYCache对象
- (nullable instancetype)initWithName:(NSString *)name;
- (nullable instancetype)initWithPath:(NSString *)path NS_DESIGNATED_INITIALIZER;
+ (nullable instancetype)cacheWithPath:(NSString *)path;

// 禁止通过下面两个方式实例化对象
- (instancetype)init UNAVAILABLE_ATTRIBUTE;
+ (instancetype)new __attribute__((unavailable("new方法不可用，请用initWithName:")));

// 通过key判断是否缓存了某个东西，第二个法是异步执行,异步回调
- (BOOL)containsObjectForKey:(NSString *)key;
- (void)containsObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, BOOL contains))block;

// 读--通过key读取缓存，第二个法是异步执行,异步回调
- (nullable id<NSCoding>)objectForKey:(NSString *)key;
- (void)objectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, id<NSCoding> object))block;

// 增、改--缓存对象(可缓存遵从NSCoding协议的对象)，第二个法是异步执行,异步回调
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key;
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key withBlock:(nullable void(^)(void))block;

// 删--删除缓存
- (void)removeObjectForKey:(NSString *)key;
- (void)removeObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key))block;
- (void)removeAllObjects;
- (void)removeAllObjectsWithBlock:(void(^)(void))block;
- (void)removeAllObjectsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                                 endBlock:(nullable void(^)(BOOL error))end;

@end
```

- YYCache使用

```
    // 0.初始化YYCache
    YYCache *cache = [YYCache cacheWithName:@"LFLdb"];
    // 1.缓存普通字符
    [cache setObject:@"哈哈哈" forKey:@"name"];
    NSString *name = (NSString *)[cache objectForKey:@"name"];
    NSLog(@"name: %@", name);
    // 2.缓存模型
    [cache setObject:(id<NSCoding>)model forKey:@"user"];
    // 3.缓存数组
    NSMutableArray *array = @[].mutableCopy;
    for (NSInteger i = 0; i < 10; i ++) {
        [array addObject:model];
    }
    // 异步缓存
    [cache setObject:array forKey:@"user" withBlock:^{
        // 异步回调
        NSLog(@"%@", [NSThread currentThread]);
        NSLog(@"array缓存完成....");
    }];
    // 延时读取
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // 异步读取
        [cache objectForKey:@"user" withBlock:^(NSString * _Nonnull key, id<NSCoding>  _Nonnull object) {
            // 异步回调
            NSLog(@"%@", [NSThread currentThread]);
            NSLog(@"%@", object);
        }];
    });
    
```    

### YYMemoryCache是内存缓存，所以存取速度非常快，主要用到两种数据结构的LRU淘汰算法
- Cache的容量是有限的，当Cache的空间都被占满后，如果再次发生缓存失效，就必须选择一个缓存块来替换掉.LRU法是依据各块使用的情况， 总是选择那个最长时间未被使用的块替换。这种方法比较好地反映了程序局部性规律
- LRU主要采用两种数据结构实现
	- 双向链表（Doubly Linked List）
	- 哈希表（Dictionary）

	
- 对一个Cache的操作无非三种：插入、替换、查找

	- 插入：当Cache未满时，新的数据项只需插到双链表头部即可
	- 替换：当Cache已满时，将新的数据项插到双链表头部，并删除双链表的尾结点即可
	- 查找：每次数据项被查询到时，都将此数据项移动到链表头部

-  移动当前节点到链表头节点


```
- (void)bringNodeToHead:(_YYLinkedMapNode *)node {
    // 当前节点已是链表头节点
    if (_head == node) return;

    if (_tail == node) {
        //**如果node是链表尾节点**

        // 把node指向的上一个节点赋值给链表尾节点
        _tail = node->_prev;
        // 把链表尾节点指向的下一个节点赋值nil
        _tail->_next = nil;
    } else {
        //**如果node是非链表尾节点和链表头节点**

        // 把node指向的上一个节点赋值給node指向的下一个节点node指向的上一个节点
        node->_next->_prev = node->_prev;
        // 把node指向的下一个节点赋值给node指向的上一个节点node指向的下一个节点
        node->_prev->_next = node->_next;
    }
    // 把链表头节点赋值给node指向的下一个节点
    node->_next = _head;
    // 把node指向的上一个节点赋值nil
    node->_prev = nil;
    // 把节点赋值给链表头节点的指向的上一个节点
    _head->_prev = node;
    _head = node;
}

```

-  移除所有缓存

```
- (void)removeAll {
    // 清空内存开销与缓存数量
    _totalCost = 0;
    _totalCount = 0;
    // 清空头尾节点
    _head = nil;
    _tail = nil;

    if (CFDictionaryGetCount(_dic) > 0) {
        // 拷贝一份字典
        CFMutableDictionaryRef holder = _dic;
        // 重新分配新的空间
        _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);

        if (_releaseAsynchronously) {
            // 异步释放缓存
            dispatch_queue_t queue = _releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else if (_releaseOnMainThread && !pthread_main_np()) {
            // 主线程上释放缓存
            dispatch_async(dispatch_get_main_queue(), ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else {
            // 同步释放缓存
            CFRelease(holder);
        }
    }
}

```

- 添加缓存

```
- (void)setObject:(id<NSCoding>)object forKey:(NSString *)key {
    if (!key) return;
    if (!object) {
        //** 缓存对象为null **

        // 删除缓存
        [self removeObjectForKey:key];
        return;
    }

    NSData *extendedData = [YYDiskCache getExtendedDataFromObject:object];
    NSData *value = nil;
    // 你可以customArchiveBlock外部归档数据
    if (_customArchiveBlock) {
        value = _customArchiveBlock(object);
    } else {
        @try {
            // 归档数据
            value = [NSKeyedArchiver archivedDataWithRootObject:object];
        }
        @catch (NSException *exception) {
            // nothing to do...
        }
    }
    if (!value) return;
    NSString *filename = nil;
    if (_kv.type != YYKVStorageTypeSQLite) {
        // ** 缓存类型非YYKVStorageTypeSQLite **
        if (value.length > _inlineThreshold) {
            // ** 缓存对象大于_inlineThreshold值则用文件缓存 **

            // 生成文件名
            filename = [self _filenameForKey:key];
        }
    }
    // 加锁
    Lock();
    // 缓存数据(此方法上面讲过)
    [_kv saveItemWithKey:key value:value filename:filename extendedData:extendedData];
    // 解锁
    Unlock();
}

```

- 写入数据库

```
- (BOOL)_dbSaveWithKey:(NSString *)key value:(NSData *)value fileName:(NSString *)fileName extendedData:(NSData *)extendedData {

    // 执行sql语句
    NSString *sql = @"insert or replace into manifest (key, filename, size, inline_data, modification_time, last_access_time, extended_data) values (?1, ?2, ?3, ?4, ?5, ?6, ?7);";
    // 所有sql执行前，都必须能run
    sqlite3_stmt *stmt = [self _dbPrepareStmt:sql];
    if (!stmt) return NO;

    // 时间
    int timestamp = (int)time(NULL);
    // 绑定参数值
    sqlite3_bind_text(stmt, 1, key.UTF8String, -1, NULL);
    sqlite3_bind_text(stmt, 2, fileName.UTF8String, -1, NULL);
    sqlite3_bind_int(stmt, 3, (int)value.length);
    if (fileName.length == 0) {
        // fileName为null时，缓存value
        sqlite3_bind_blob(stmt, 4, value.bytes, (int)value.length, 0);
    } else {
        // fileName不为null时，不缓存value
        sqlite3_bind_blob(stmt, 4, NULL, 0, 0);
    }
    sqlite3_bind_int(stmt, 5, timestamp);
    sqlite3_bind_int(stmt, 6, timestamp);
    sqlite3_bind_blob(stmt, 7, extendedData.bytes, (int)extendedData.length, 0);

    // 执行操作
    int result = sqlite3_step(stmt);
    if (result != SQLITE_DONE) {
        //** 未完成执行数据库 **

        // 输出错误logs
        if (_errorLogsEnabled) NSLog(@"%s line:%d sqlite insert error (%d): %s", __FUNCTION__, __LINE__, result, sqlite3_errmsg(_db));
        return NO;
    }
    return YES;
}

```



