---
title: 基础知识拾遗-GCD串行并发
date: 2019-04-19 14:13:41
tags: 多线程
categories: iOS
---

### 前言
总结本篇博客内容：

1. 串行并发的基本概念
2. 同步异步的基本概念
3. 死锁的必要条件
4. 同步(串行\并发)队列是否会创建新的子线程执行
5. 异步(串行\并发)队列是否会创建新的子线程执行<!--more-->

### 串行&并发（队列）

#### 串行队列
> 串行队列：比如这里的dispatch_get_main_queue。这个队列中所有任务，一定按照FIFO(先来后到的顺序)执行。不仅如此，还可以保证在执行某个任务时，在它前面进入队列的所有任务肯定执行完了。对于每一个不同的串行队列，系统会为这个队列建立唯一的线程来执行代码。

#### 并发队列
> 比如使用dispatch_get_global_queue。这个队列中的任务也是按照FIFO(先来后到的顺序)开始执行，注意是开始，但是它们的执行结束时间是不确定的，取决于每个任务的耗时。异步并发队列中的任务：GCD会动态分配多条线程来执行。具体几条线程取决于当前内存使用状况，线程池中线程数等因素。

### 同步&异步
#### 同步
> 同步执行：比如这里的dispatch_sync，这个函数会把一个block加入到指定的队列中，而且会一直等到执行完blcok，这个函数才返回。因此在block执行完之前，调用dispatch_sync方法的线程是阻塞的。

#### 异步
> 异步执行：一般使用dispatch_async，这个函数也会把一个block加入到指定的队列中，但是和同步执行不同的是，这个函数把block加入队列后不等block的执行就立刻返回了。

### 死锁
> 所谓死锁，通常指有两个线程T1和T2都卡住了，并等待对方完成某些操作。T1不能完成是因为它在等待T2完成。但T2也不能完成，因为它在等待T1完成。于是大家都完不成，就导致了死锁（DeadLock）。

#### 死锁的必要条件
>1. 互斥条件：一个资源每次只能被一个进程使用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
4. 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

##### 经典的死锁代码
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        dispatch_sync(dispatch_get_main_queue(), ^(void){
            NSLog(@"这里死锁了");
        });
    }
    return 0;
}
```
##### 分析
首先明确的是：执行这个dispatch_get_main_queue队列的是主线程。执行了dispatch_sync函数后，将block添加到了main_queue中，同时调用dispatch_syn这个函数的线程（也就是主线程）被阻塞，等待block执行完成，而执行主线程队列任务的线程正是主线程，此时他处于阻塞状态，所以block永远不会被执行，因此主线程一直处于阻塞状态。因此这段代码运行后，并非卡在block中无法返回，而是根本无法执行到这个block。

### 验证是否会开辟子线程执行任务
#### 异步串行队列
```
dispatch_queue_t que = dispatch_queue_create("com.my", DISPATCH_QUEUE_SERIAL);
for (int i = 0; i < 10; i++) {
        dispatch_async(que, ^{
            if ([NSThread isMainThread]) {
                NSLog(@"是");
            } else {
                NSLog(@"不是");
            }
            NSLog(@"%@", [NSThread currentThread]);
        });
    }
    // 具体的log就不贴出了，可直接复制代码运行查看
```

#### 异步并发队列
```
dispatch_queue_t que = dispatch_queue_create("com.my", DISPATCH_QUEUE_CONCURRENT);
```

#### 同步串行队列
```
dispatch_queue_t que = dispatch_queue_create("com.my", DISPATCH_QUEUE_SERIAL);
for (int i = 0; i < 20; i++) {
        dispatch_sync(que, ^{
            if ([NSThread isMainThread]) {
                NSLog(@"是");
            } else {
                NSLog(@"不是");
            }
            NSLog(@"%@", [NSThread currentThread]);
        });
    }
```

#### 同步并发队列
```
dispatch_queue_t que = dispatch_queue_create("com.my", DISPATCH_QUEUE_CONCURRENT);
```

### 总结
1. 异步并发队列会根据当前环境和线程池状态另开辟多个子线程执行
2. 异步串行队列会创建一个子线程执行队列任务
3. 同步串行队列不会创建子线程执行
4. 同步并发队列不会创建子线程执行
5. ⭐️在某一个串行队列中，同步的向这个串行队列添加任务会产生死锁



