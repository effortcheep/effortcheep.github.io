---
title: RunLoop在实际开发中的应用
date: 2020-05-26 20:47
tags:
	- iOS
	- RunLoop
---
## RunLoop在实际开发中的应用
### 控制线程生命周期(线程保活)
```objective-c
- (void)viewDidLoad {
    [super viewDidLad];
    NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
    [thread start];
}
- (void)run {
    NSLog(@"run");
}
// run -> thread deallo

// 线程保活
- (void)viewDidLoad {
    [super viewDidLoad];
    // initWithTarget:self 会强引用控制器 导致循环引用
    // self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];

    self.stopped = false;

    __weak typeof(self) weakSelf = self;
    self.thread = [[NSThread alloc] initWithBlock:^{
        NSLog(@"run");
        // [NSRunLoop currentRunLoop] 获取线程就是创建线程
        // 需要往RunLoop添加Source/Timer/Observer
        [[NSRunLoop currentRunLoop] addPort:[[NSPort alloc] init] forMode:NSDefaultRunLoopMode];

        /*
         it runs the receiver in the NSDefaultRunLoopMode by repeatedly invoking runMode:beforeDate:
         In other words, this method effectively begins an infinite loop that processes data from the run loop’s input sources and timers.
         [[NSRunLoop currentRunLoop] run] 相当于
         while (1) {
             [NSRunLoop currentRunLoop] runMode:<#(nonnull NSRunLoopMode)#> beforeDate:<#(nonnull NSDate *)#>
         }
         */
        // NSRunLoop 的 run 方法是无法停止的，他专门用于开启一个用不销毁的线程(RunLoop)
        // [[NSRunLoop currentRunLoop] run];

        //线程保活
        while (weakSelf && !weakSelf.isStopped) {
            [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
        }

        NSLog(@"end");
    }];
    [self.thread start];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 最后一个参数为 YES 会等待 线程中 test 方法执行完打印 123
    // 最后一个参数为 NO 会直接打印 123
    if (!self.thread) return;
    [self performSelector:@selector(test) onThread:self.thread withObject:nil waitUntilDone:NO];
    NSLog(@"123");
}

- (void)test {
    NSLog(@"test");
}

- (void)stop {
    if (!self.thread) return;
    // waitUntilDone 只能为 YES
    // NO 会造成野指针 控制销毁和子线程 stop方法同时进行 控制器坏内存
    [self performSelector:@selector(stopThread) onThread:self.thread withObject:nil waitUntilDone:YES];
}

// 用于停止子线程的 RunLoop
- (void)stopThread {
    self.stopped = true;
    CFRunLoopStop(CFRunLoopGetCurrent());
    self.thread = nil;
}

- (void)dealloc {
    [self stop];
    // dealloc 执行完毕 weakSelf 为空
}
```
### 解决NSTimer在滑动时停止工作问题
+ NSDefaultRunLoopMode 和 UITrackingRunLoopMode 才是真正存在的模式
+ NSRunLoopCommonModes 并不是真正的一种 RunLoop模式，只是一个标记，能在_commonModes(RunLoop结构体中)数组中存放的模式中运行
    + 如果 timer 是 NSRunLoopCommonModes 该模式下的，就会存放在_commonModeItems里(RunLoop结构体中)

### 监控应用卡顿
### 性能优化
