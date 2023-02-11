---
title: KVC原理
date: 2020-05-25 16:43
tags:
	- iOS
	- KVC
---

## KVC
**KVC(Key-Value Coding)键值编码**是利用**NSKeyValueCoding**非正式协议实现的一种机制，对象采用过这种机制来提供对其属性的间接访问
+ **KVC**通过对**NSObject**的扩展来实现，所有继承了**NSObject**的类都可以使用**KVC**
+ **NSArray**、**NSDictionary**、**NSSet**、**NSOrderedSet**也遵循**KVC**协议
+ 除了少数类型(结构体)以外都可以使用**KVC**
```objective-c
// 通过 key 设值
- (void)setValue:(nullable id)value forKey:(NSString *)key;
// 通过 key 取值
- (nullable id)valueForKey:(NSString *)key;
// 通过 keyPath 设值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;
// 通过 keyPath 取值
- (nullable id)valueForKeyPath:(NSString *)keyPath;
```
**NSKeyValueCoding**类别的其它方法
```objective-c
// 默认YES。如果返回YES，如果没有找到 set<Key> 方法的话， 会按照_key，_isKey，key，isKey的顺序搜索成员变量，返回NO则不会搜索
+ (BOOL)accessInstanceVariablesDirectly;
// 键值验证，可以通过高方法检验键值的正确性
- (BOOL)validateValue:(inout id _Nullable * _Nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;
// 如果key不存在，并且没有搜索到和key相关的字段，就会调用此方法，抛出异常。两个方法分别对应 get 和 set
- (nullable id)valueForUndefineKey:(NSString *)key;  // get
- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key; // get
// setValue 方法传 nil 时调用此方法
// 当且仅当 NSNumber 和 NSValue 类型是才会调用此方法
- (void)setNilValueForKey:(NSString *)key;
// 一组 key 对应的value，将其转成字典返回，可用于将 Model 转成字典
- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
```

## 集合操作符
+ 聚合操作符
    + **@avg**: 返回操作对象指定属性的平均值
    + **@count**: 返回操作对象指定属性的个数
    + **@max**: 返回操作对象指定属性的最大值
    + **@min**: 返回操作对象指定属性的最小值
    + **@sum**: 返回操作对象指定属性值之和
+ 数组操作符
    + **@distinctUnionOfObjects**: 返回操作对象指定属性的集合--去重
    + **@unionOfObjects**: 返回操作对象指定属性的集合
+ 嵌套操作符
    + **@distinctUnionOfArrays**: 返回操作对象(嵌套集合)指定属性的集合--去重，返回的是**NSArray**
    + **@unionOfArrays**: 返回操作对象(集合)指定属性的集合
    + **@distinctUnionOfSets**: 返回操作对象(嵌套集合)指定属性的集合--去重，返回的是**NSSet**


## 底层原理
#### ***Setter*** 赋值方法
1. 按 **set<Key>**:、**_set<Key>**: 顺序查找对象中是否有对应的方法
    + 找到了直接调用方法赋值
    + 没有找到跳到第2步
2. 通过判断**accessInstanceVariablesDirectly**返回值
    + **YES** 按照 **_<key>**、**_is<Key>**、**<key>**、**is<Key>**的顺序查找成员变量，找到了赋值；没找到跳到第3步
    + **is<Key>** 跳到第3步
3. 调用 **setValue: forUndfinedKey:** 默认情况下抛出异常，子类可以重写该方法避免崩溃
<!-- ![](./_image/2020-05-25/2020-05-25-17-15-05.png) -->

#### ***Getter*** 取值方法
1. 按照 **get<Key>**、**<key>**、**is<Key>**、**<key>** 顺序查找对象中是否有对应的方法
2. 查找是否有**countOf<Key>**和**objectIn<Key>AtIndex:**方法(对应于**NSArray**类定义的原始方法)以及**<key>AtIndexes:**方法(对应于**NSArray**方法**objectsAtIndexes:**)
    + 如果找到其中的第一个(**countOf<Key>**)，再找到其他两个中的至少一个，则创建一个响应所有**NSArray**方法的代理集合对象，并返回该对象(即要么是**countOf<Key> + objectIn<Key>AtIndex:**，要么是**+ <key>AtIndexes:，要么是countOf<Key> + objectIn<Key>AtIndex: + <key>AtIndexes:**)
    + 如果没有找到，跳转到第3步
3. 查找名为**counOf<Key>**、**enumeratorOf<Key>**和**memberOf<Key>**这三个方法(对应于NSSet类定义的原始方法)
    + 如果找到这三个方法，则创建一个响应所有**NSSet**方法的代理集合对象，并返回该对象
    + 如果没有找到，跳转到第4步
4. 判断**accessInstanceVariablesDirectly**
    + 为**YES**时按照**_<key>**、**_is<Key>**、**<key>**、**is<Key>**的顺序查找成员变量，找到了就取值
    + 为**NO**时跳转第6步
5. 判断取出的属性值
    + 属性值是对象，直接返回
    + 属性值不是对象，但是可以转化为**NSNumber**类型，则将属性值转化为**NSNumber**类型返回
    + 属性值不是对象，也不能转化为**NSNumber**类型，则将属性值转化为**NSValue**类型返回
6. 调用**valueForUndefinedKey:**默认情况下抛出异常，子类可以重写该方法避免崩溃

<!-- ![](./_image/2020-05-25/2020-05-25-17-17-58.png) -->



