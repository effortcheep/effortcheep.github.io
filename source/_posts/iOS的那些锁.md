---
title: iOS的那些锁
date: 2020-05-26 23:21
tags:
  - iOS
	- LOCK
---

## 自旋锁

  自旋锁的线程会一直处于等待状态(忙等待)不会进入休眠 效率高

## 互斥锁

  在线程获取锁但没有获取到时，线程会进入休眠状态，等锁被释放时线程会被唤醒

## 互斥锁

  防止两个线程同时对用一个公共资源进行读写的机制。当获取锁操作失败时，线程会进入睡眠，等待锁释放时被唤醒

## 互斥锁又分为
+ 递归锁 可重入锁，同一个线程在锁释放前可再次获取锁，即可递归调用
+ 非递归锁 不可重入，必须等锁释放后才能再次获取锁

## 自旋锁
线程反复监测随便啦是否可用。由于线程在这一过程中保持执行，因此是一种忙等待。一但获取了自旋锁，线程会一直保持该锁，知道显示释放自旋锁
自旋锁避免了进程上下文的调度开心，因此多余线程只会阻塞很短时间的场合是有效的

## 自旋锁
1. OSSpinLock
+ OSSpinLock 已被废弃
     自旋锁之所以不安全，是因为自旋锁由于获取锁时，线程一直处于忙等待状态，造成了任务的优先级反转，而OSSpinLock忙等待的实际就可能造成高优先级一直running等待，占用CPU时间片，而低优先级任务无法抢占时间片，变成迟迟完不成，不释放锁的情况。
2. automic
+ 只能保证setter、getter方法的线程安全，并不能保证数据安全

3. 读写锁
+ 读写锁实际是一种特殊的自旋锁，他把对共享资源的访问者划分成读者和写者，读者只对共享资源进行访问，写者则需要对共享资源进行写操作。这种锁相对于自旋锁而言，能提高并发性，因为在多处理器系统中，它允许同时有多个读者来访问共享资源，最大可能的读者数为实际的CPU数
	+ 写者是排他性的，一个读写锁同时只能有一个写者或者多个读者(与CPU数相关)，但不能同时既有读者又有写者。在读写锁保持器件也是抢占时效的
	+ 如果读写锁当前没有读者，也没有写者，那么写者可以立即获得写锁，否则它必须自旋在那里，直到没有任何写者或者读者。如果读写锁没有写者，那么读者可以立即获得该读写锁，否则读者必须自旋在那里，直到写者释放该读写锁

```objective-c
#import <pthread.h>
// 全局声明读写锁
pthread_rwlock_t lock;
// 初始化读写锁
pthread_rwlock_init(&lock, NULL);
// 读操作-加锁
pthread_rwlock_rdlock(&lock);
// 读操作-尝试加锁
pthread_rwlock_tryrdlock(&lock);
// 写操作-加锁
pthread_rwlock_wrlock(&lock);
// 写操作-尝试加锁
pthread_rwlock_trywrlock(&lock);
// 解锁
pthread_rwlock_unlock(&lock);
// 释放锁
pthread_rwlock_destroy(&lock);
```
平时很少直接使用读写锁pthread_rwlock_t，更多的是采用其他方式，例如使用栅栏函数完成读写锁的需求

## 互斥锁

1. pthread_mutex
+ pthread_mutex就是互斥锁本身 --- 当锁被占用，而其他线程申请锁时，不是使用忙等，而是阻塞线程并睡眠
```objective-c
	#import <pthread.h>
	// 全局声明互斥锁
	pthread_mutex_t _lock;
	// 初始化互斥锁
	pthread_mutex_init(&_lock, NULL);
	// 加锁
	pthread_mutex_lock(&_lock);
	// 做需要线程安全操作
	// ...
	// 解锁
	pthread_mutex_unlock(&_lock);
	// 释放锁
	pthread_mutex_destroy(&_lock);
```

2. @synchronized
+ 递归锁
+ @synchronized 可能是日常开发中用的比较多的一种互斥锁，但并不是在任意场景下都能使用@synchronized，且它的性能较低
+ 性能低是因为的层增删改查消耗了大量新能
 + 底层维护一个`SyncData`哈希链表
```c
typedef struct SyncData {
		struct SyncData* nextData;   // 指向下一个
		DisguisedPtr<objc_object> object;
		int32_t threadCount;  // number of THREADS using this block
		recursive_mutex_t mutex;
} SyncData;
```

```objective-c
// Begin synchronizing on 'obj'.
// Allocates recursive mutex associated with 'obj' if needed.
// Returns OBJC_SYNC_SUCCESS once lock is acquired.
int objc_sync_enter(id obj)
{
		int result = OBJC_SYNC_SUCCESS;

		if (obj) {
				SyncData* data = id2data(obj, ACQUIRE);
				assert(data);
				data->mutex.lock();
		} else {
				// @synchronized(nil) does nothing
				if (DebugNilSync) {
						_objc_inform("NIL SYNC DEBUG: @synchronized(nil); set a breakpoint on objc_sync_nil to debug");
				}
				objc_sync_nil();
		}

		return result;
}

// End synchronizing on 'obj'.
// Returns OBJC_SYNC_SUCCESS or OBJC_SYNC_NOT_OWNING_THREAD_ERROR
int objc_sync_exit(id obj)
{
		int result = OBJC_SYNC_SUCCESS;

		if (obj) {
				SyncData* data = id2data(obj, RELEASE);
				if (!data) {
						result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
				} else {
						bool okay = data->mutex.tryUnlock();
						if (!okay) {
								result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
						}
				}
		} else {
				// @synchronized(nil) does nothing
		}

		return result;
}
```

3. NSLock
+ NSLokc是对互斥锁(pthread_mutex)的简单封装
```objective-c
   - (void)test {
       self.testArray = [NSMutableArray array];
       NSLock *lock = [[NSLock alloc] ini];
       for (int i = 0; i < 20000; i++) {
           dispatch_async(dispatch_get_global_queue(0, 0), ^{
               [lock lock];
               self.testArray = [NSMutableArray array];
               [lock unlock];
           });
       }
   }
```
+ 注意事项
 使用互斥锁NSLock异步并发调用block快，block快内部递归调用自己
   ```objective-c
   - (void)test {
       NSLock *lock = [[NSLock alloc] init];
       dispatch_async(dispatch_get_global_queue(0, 0), ^{
           static void (^block)(int);

           block = ^(int value) {
               NSLog(@"加锁前");
               [lock lock];
               NSLog(@"加锁后");
               if (value > 0) {
                   NSLog(@"value--%d", value);
                   block(value - 1);
               }
               [lock unlock];
           };
           block(10);
       });
   }
   // 加锁前 -> 加锁后 -> value--10 -> 加锁前
```
   原因：互斥锁在递归调用是会造成堵塞，并非死锁
    + 第一次加锁之后还没有出锁就进行递归调用
    + 第二次加锁就堵塞了线程

   解决方案：使用递归锁 NSRecursiveLock 替换 NSLock

4. NSRecursiveLock
+ NSRecursiveLock使用和NSLock类似 使用 NSRecursiveLock 可以解决上面问题
```objective-c
   - (void)test {
       NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
       dispatch_async(dispatch_get_global_queue(0, 0), ^{
           static void (^block)(int);

           block = ^(int value) {
               NSLog(@"加锁前");
               [lock lock];
               NSLog(@"加锁后");
               if (value > 0) {
                   NSLog(@"value--%d", value);
                   block(value - 1);
               }
               [lock unlock];
           };
           block(10);
       });
   }
   // 加锁前 -> 加锁后 -> value--10 -> 加锁前 -> 加锁后 -> value--9 -> 加锁前 -> 加锁后 -> ...
```
+ 注意事项
 递归锁在使用时需要注意思索问题 -- 前后代码互相等待便会产生思索

```objective-c
   - (void)test {
       NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
       for (int i = 0; i < 10; i++) {
           dispatch_async(dispatch_get_global_queue(0, 0), ^{
               static void (^block)(int);

               block = ^(int value) {
                   NSLog(@"加锁前");
                   [lock lock];
                   NSLog(@"加锁后");
                   if (value > 0) {
                       NSLog(@"value--%d", value);
                       block(value - 1);
                   }
                   [lock unlock];
               };
               block(10);
           });
       }
   }
```
  `Crash`提示野指针错误
   原因：for 循环在block内部对同一个对象进行了多次锁操作，知道这个资源上挂着N把锁，最后大家都无法一次性解锁---找不到解锁的出口
        即 线程1中加锁1、同时线程2中加锁2 -> 解锁1等待解锁2 -> 解锁2等待解锁1 -> 无法结束解锁 -- 形成思索

   解决：可以采用使用缓存的@synchronized，因为他对对象进行锁操作，会先从缓存查找是否有所syncData存在。如果有，直接返回而不加锁，保证所锁的唯一性


5. dispatch_semaphore_t
+ 信号量
+ dispatch_semaphore_create() 创建信号量
+ dispatch_semaphore_wait() 等待信号量，信号量减1。 当信号量 < 0 时会阻塞当前线程，根据传入的等待时间决定接下来的操作--如果永久等地啊将等到信号(signal) 才执行下去
+ dispatch_semaphore_signal() 释放信号量，信号量加1。 当信号量 >= 0 会执行 wait 之后的代码

   要求按顺序执行

```objective-c
   - (void)test {
       dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);

       for (int i = 0; i < 10; i++) {
           dispatch_async(queue, ^ {
               NSLog("当前%d----线程%@", i, [NSThread currentThread]);
           });
           // 使用栅栏函数
           // dispatch_barrier_async(queue, ^{});
       }
   }

   - (void)dispatch_semaphore_test {
       dispatch_semaphore_t sem = dispatch_semaphore_create(0);
       dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);

       for (int i = 0; i < 10; i++) {
           dispatch_async(queue, ^ {
               NSLog("当前%d----线程%@", i, [NSThread currentThread]);
               // 打印任务结束后信号量解锁
               dispatch_semaphore_signal(sem);
           });
           // 加锁
           // dispatch_semaphore_wait(sem);
       }
   }
```
   + 创建信号量时传入值为1时，可以通过两次才堵塞
   + 传入值为2时，可以通过三次才堵塞


6. NSCondition
+ NSCondition 是一个条件锁，线程1需要等到条件1满足才会往下走，否则就会堵塞等待，直至条件满足
```swift
   open class NSCondition: NSObject, NSLocking {
       internal var mutex = _MutexPointer.allocate(capacity: 1)
       internal var cond = _ConditionVariablePointer.allocate(capacity: 1)

       public override init() {
           pthread_mutex_init(mutex, nil)
           pthread_cond_init(cond, nil)
       }

       deinit {
           pthread_mutex_destroy(mutex)
           pthread_cond_destroy(cond)
       }

       open func lock() {
           pthread_mutex_lock(mutex)
       }

       open func unlock() {
           pthread_mutex_unlock(mutex)
       }

       open func wait() {
           pthread_cond_wait(cond, mutex)
       }

       open func wait(until limit: Date) -> Bool {
           guard var timeout = timeSpecFrom(date: limit) else {
               return false
           }
           return pthread_cond_timedwait(cond, mutex, &timeout) == 0
       }

       open func signal() {
           pthread_cond_signal(cond)
       }

       open func broadcast() {
           pthread_cond_broadcast(cond) // wait  signal
       }

       open var name: String?
   }
```

   + NSCondition 是对 mutex 和 cond 的一种封装(cond就是用于访问和操作特定类型数据的指针)
   + wait 操作会阻塞线程，使其进入休眠状态，直至超时
   + signal 操作是唤醒一个正在休眠等待的线程
   + broadcast 会唤醒所有正在等待的线程


7. NSConditionLock
+ NSCondition + Lock
```swift
   open class NSConditionLock : NSObject, NSLocking {
       internal var _cond = NSCondition()
       internal var _value: Int
       internal var _thread: _swift_CFThreadRef?

       public convenience override init() {
           self.init(condition: 0)
       }

       public init(condition: Int) {
           _value = condition
       }

       open func lock() {
           let _ = lock(before: Date.distantFuture)
       }

       open func unlock() {
           _cond.lock()
           _thread = nil
           _cond.broadcast()
           _cond.unlock()
       }

       open var condition: Int {
           return _value
       }

       open func lock(whenCondition condition: Int) {
           let _ = lock(whenCondition: condition, before: Date.distantFuture)
       }

       open func `try`() -> Bool {
           return lock(before: Date.distantPast)
       }

       open func tryLock(whenCondition condition: Int) -> Bool {
           return lock(whenCondition: condition, before: Date.distantPast)
       }

       open func unlock(withCondition condition: Int) {
           _cond.lock()
           _thread = nil
           _value = condition
           _cond.broadcast()
           _cond.unlock()
       }

       open func lock(before limit: Date) -> Bool {
           _cond.lock()
           while _thread != nil {
               if !_cond.wait(until: limit) {
                   _cond.unlock()
                   return false
               }
           }
           _thread = pthread_self()
           _cond.unlock()
           return true
       }

       open func lock(whenCondition condition: Int, before limit: Date) -> Bool {
           _cond.lock()
           while _thread != nil || _value != condition {
               if !_cond.wait(until: limit) {
                   _cond.unlock()
                   return false
               }
           }
           _thread = pthread_self()
           _cond.unlock()
           return true
       }

       open var name: String?
   }
```
+ NSConditionLock 是 NSCondition加线程数的封装
+ NSConditionLock 可以设置锁条件，而NSCondition只是无脑的通知信号

8. os_unfair_lock
+ 如果等待锁的线程优先级比较高，它会一直占用着CPU资源，优先级低的线程就无法释放锁
+ 由于 OSSpinLock自旋锁的BUG，替代方案是内部封装了os_unfair_lock，而os_unfair_lock在加锁是会处于休眠状态，而不是自旋锁的等待状态

## 总结
+ OSSpinLock不在安全，底层使用os_unfair_lock替代
+ atomic只能保证setter、getter时线程安全，所以更多的时候使用nonatiomic修饰
+ 读写锁更多使用栅栏函数来实现
+ @synchronized在底层维护了一个哈希链表进行data的存储，使用recursive_mutex_t进行加锁
+ NSLock、NSRecursiveLock、NSCondition和NSConditionLock底层都是对pthread_mutex的封装
+ NScondition和NSConditionLock是条件锁，当满足某一个条件时才能执行操作，和信号量dispatch_semaphore类似
+ 普通场景下涉及到线程安全，可以使用NSLock
+ 循环调用时用 NSRecuriveLock
+ 循环调用且有线程影响是，注意死锁，有死锁问题使用@synchronized







