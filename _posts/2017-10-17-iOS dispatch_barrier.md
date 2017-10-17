---
layout: post
title: "dispatch _ barrier _ sync和dispatch _ barrier _ async 的区别"
---

#dispatch _ barrier _ sync和dispatch _ barrier _ async 的区别

前几天在看SDWebImage源码的时候看到了这两个API

```objc

dispatch_barrier_sync(self.barrierQueue, ^{
        BOOL first = NO;
        if (!self.URLCallbacks[url]) {
            self.URLCallbacks[url] = [NSMutableArray new];
            first = YES;
        }
        // Handle single download of simultaneous download request for the same URL
        NSMutableArray *callbacksForURL = self.URLCallbacks[url];
        NSMutableDictionary *callbacks = [NSMutableDictionary new];
        if (progressBlock) callbacks[kProgressCallbackKey] = [progressBlock copy];
        if (completedBlock) callbacks[kCompletedCallbackKey] = [completedBlock copy];
        [callbacksForURL addObject:callbacks];
        self.URLCallbacks[url] = callbacksForURL;
        if (first) {
            createCallback();
        }
    }); 
    
```
    
 翻译过来大家一般都叫做栅栏函数,在一个线程队列里如果希望先完成某些任务之后,再执行之后的任务,就可以使用这个函数来操作,这是barrier的意思,但是为什么还有一个sync 和 async呢?
 举个例子
 
```objc   
 
 dispatch_queue_t queue = dispatch_queue_create("test.queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        NSLog(@"1111");
    });
    dispatch_async(queue, ^{
        NSLog(@"2222");
    });
    dispatch_async(queue, ^{
        NSLog(@"3333");
    });
    dispatch_barrier_async(queue, ^{

        for (int i = 0; i <5000000; i ++) {
            if (i == 1000000) {
                NSLog(@"1000000");
            }
            else if (i == 2000000) {
                NSLog(@"2000000");
            }
            else if (i == 3000000) {
                NSLog(@"3000000"); 
            }
        }
    });
    NSLog(@"after barrier");
    NSLog(@"888888888");
    NSLog(@"99999999");
    dispatch_async(queue, ^{
        NSLog(@"4444");
    });
    dispatch_async(queue, ^{
        NSLog(@"5555");
    });
    dispatch_async(queue, ^{
        NSLog(@"6666");
    });
    
```
[4042:758675] 888888888

[4042:758953] 2222

[4042:758951] 3333

[4042:758952] 1111

[4042:758675] 99999999

[4042:758952] 1000000

[4042:758952] 2000000

[4042:758952] 3000000

[4042:758952] 4444

[4042:758951] 5555

[4042:758960] 6666

*****************修改为sync之后的打印*****************

[4084:768014] 2222

[4084:768015] 1111

[4084:768020] 3333

[4084:767596] 1000000

[4084:767596] 2000000

[4084:767596] 3000000

[4084:767596] 888888888

[4084:767596] 99999999

[4084:768015] 4444

[4084:768021] 5555

[4084:768020] 6666


 两个函数都能够保证 在queue中任务的执行barrier前面的先执行完成,然后再执行barrier里面和后面的任务,区别在于,888888 和 999999这两个的执行时机,在使用sync的时候 888888和999999的执行永远是在barrier中的任务执行完成之后再执行因为sync就是为了阻塞当前队列(当前为主队列). 而async则不会阻塞当前队列,所以在第一次的打印中888888和99999的执行和barrier就无关了,任何时候都可能打印.
 
 
 再回到SD的源码中,使用sync的原因是因为为了在当前线程中(异步)线程中同步的去执行当前的代码.
