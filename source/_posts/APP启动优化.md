---
title: APP启动优化
date: 2020-05-28 20:55
---

## APP的启动
### APP的启动分为2种
+ 冷启动(Cold Launch)：从零开始启动APP
+ 热启动(Warm Launch)：APP已经在内存中，再次点击图标启动APP

### APP的启动时间优化，主要是针对冷启动进行优化
+ 通过添加环境变量可以打印出APP的启动时间分析(Edit scheme -> Run -> Arguments)
    + DYLD_PRINT_STATISTICS设置为1
    + DYLD_PRINT_STATISTICS_DETAILS设置为1，查看跟详细的信息

## APP的冷启动可以概况为3大阶段

### dyld
dyld(dynamic link editor)，Apple的动态链接器，可以用来装载Mach-O文件(可执行文件、动态库等)
启动APP，dyld做的事情
+ 装载APP的可执行文件，同时会递归加载所有依赖的动态库
+ 当dyld把可执行文件、动态库都装载完毕后，会通知Runtime进行下一个处理
### runtime
+ 调用map_images进行可执行文件内容的解析和处理
+ 在load_images中调用call_load_methods，调用所有Class和Category的+load方法
+ 进行各种objc结构的初始化(注册Objc类、初始化类对象等等)
+ 调用C++静态初始化器和__attribute__((constructor))修饰的函数

到此为止，可执行文件和动态库中所有的符号(Class，Protocol，Selector，IMP，...)都已经按格式成功加载到内从中，被runtime所管理

### main
APP的启动由dyld主导，将可执行文件加载到内存，顺便加载所有依赖的动态库
并由runtime负责加载成objc定义的结构
所有初始化结束后，dyld就会调用Main函数


## 优化

### dyldb
+ 减少动态库、合并一些动态库
+ 减少Objc类、分类的数量、减少Selector数量
+ 减少C++虚函数数量
+ swift尽量使用struct

### runtime
+ 用 +initialize方法和dispatch_once取代所有的__attribute__((constructor))、C++静态构造器、Object的+load

### main
+ 尽可能将一些操作延迟，不要全部放在finishLaunching方法中
