---
title: 对Strong-Weak Dance的思考
date: 2019-03-25 18:14:09
tags: 面试题
categories: iOS
---

在使用 Block 时，除了使用 __weak 修饰符避免循环引用外，还有一点经常容易忘记。苹果把它称为：“Strong-Weak Dance”。<!--more-->

### 背景
这是一种 强引用 --> 弱引用 --> 强引用 的变换过程。

```
__weak typeof(self) wself = self;
self.completionHandler = ^(NSInteger result) {
[wself.property removeObserver: wself forKeyPath:@"pathName"];
};

```

这种写法可以避免循环引用，但是我们要考虑这样的问题：

假设 block 被放在子线程中执行，而且执行过程中 self 在主线程被释放了。由于 wself 是一个弱引用，因此会自动变为 nil。而在 KVO 中，这会导致崩溃。

解决以上问题的方法很简单，新增一行代码即可：

```
__weak typeof(self) wself = self;
self.completionHandler = ^(NSInteger result) {
__strong __typeof(wself) sself = wself; // 强引用一次
[sself.property removeObserver: sself forKeyPath:@"pathName"];
};
```

这样一来，self 所指向对象的引用计数变成 2，即使主线程中的 self 因为超出作用于而释放，对象的引用计数依然为 1，避免了对象的销毁。

### Q&A

1. Q：下面这行代码，将一个弱引用的指针赋值给强引用的指针，可以起到强引用效果么？

```
__strong __typeof(wself) sself = wself;
```

A：会。引用计数描述的是对象而不是指针，sself强引用wself指向的那个对象，因此对象的引用计数加一。

2. Q：block 内部定义了sself，会不会因此强引用了 sself？

A：不会。block只有捕获外部变量才会引用，具体原理可查看[霜神的这篇博客](https://www.jianshu.com/p/ee9756f3d5f6)。

3. Q：如果在 block 内部没有强引用，而是通过 if 判断，是不是也可以，比如这样写：

```
__weak MyViewController *wself = self;
wself.completionHandler = ^(NSInteger result) {
if (wself) { // 只有当 wself 不为 nil 时，才执行以下代码
[wself.property removeObserver: wself forKeyPath:@"pathName"];
}
};
```

A：不可以！考虑到多线程执行，也许在判断的时候，self 还没释放，但是执行 self 里面的代码时，就刚好释放了。

4. Q：那按照这个说法，block 内部强引用也没用啊。也许 block 执行以前，self 就释放了。

A：有用！如果在 block 执行以前，self 就释放了，那么 block 的引用计数降为 0，所以自己就会被释放。这样它根本就不会被执行。另外，如果执行一个为 nil 的闭包会导致崩溃。

5. Q：如果在执行 block 的过程中，block 被释放了怎么办？
A：简单来说，block 还会继续执行，但是它捕获的指针会具有不确定的值。[点我](https://stackoverflow.com/questions/12272783/what-happens-when-a-block-is-set-to-nil-during-its-execution)

### 引申面试题

#### 执行一个为nil的block为什么会崩溃
在开发时，调用闭包我们都会先if判空，但很少问为什么

```
- (void)doSomething:(void(^)())finished{
    NSLog(@"I am doing something...");
    if(finished){
        finished();
    }
}
```
而假如再没有判空的前提下外界直接像下面这样调用

```
[self doSomething:nil];
```

上面的代码则会发生__EXC_BAD_ACCESS__崩溃信息。通常这样的崩溃原因一般就是：调用了已释放的内存空间或者重复释放某个地址空间。具体原理如下：

```
// Block在OC中的实现如下
struct Block_layout {
    void *isa;
    int flags;
    int reserved;
    void (*invoke)(void *, ...);
    struct Block_descriptor *descriptor;
    /* Imported variables. */
};
```

block是指向这个结构体的指针，invoke就是指向具体实现的函数指针。当block被调用时，程序最终会跳转到指针指向的代码区。而当block为nil时，程序会试图读取地址信息，而这个地址什么都没有，于是抛出异常。

所以我们在使用block时，先去非空判断。一种优雅的方式参考[sunnyxx](http://blog.sunnyxx.com)的某篇博文，具体忘记了

```
!_block?:_block();
```

#### 给nil发消息是否会崩溃

首先，OC中向nil发消息是不会崩溃的。

> 因为OC的函数调用都是通过objc_msgSend进行消息发送来实现的，相对于C和C++来说，对于空指针的操作会引起Crash的问题，而objc_msgSend会通过判断self来决定是否发送消息，如果self为nil，那么selector也会为空，直接返回，所以不会出现问题。视方法返回值，向nil发消息可能会返回nil(返回值为对象)、0（返回值为一些基础数据类型）或0X0（返回值为id）等。但是对[NSNull 
null]对象发送消息时，是会crash的，因为这个NSNull类只有一个null方法。
当然，如果一个对象已经被释放了（引用计数为0了），那么这个时候再去调用方法肯定是会Crash的，因为这个时候这个对象就是一个野指针（指向僵尸对象（对象的引用计数为0，指针指向的内存已经不可用）的指针）了，安全的做法是释放后将对象重新置为nil，使它成为一个空指针。

### 参考
[bestswifter的对 Strong-Weak Dance的思考](https://juejin.im/post/5a309c2751882531ba10ed9d)
