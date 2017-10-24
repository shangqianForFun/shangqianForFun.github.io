---
layout: post
title: "[摘] Strong-Weak Dance"
---

# [摘] Strong-Weak Dance

在使用 Block 时，除了使用 __weak 修饰符避免循环引用外，还有一点经常容易忘记。苹果把它称为：“Strong-Weak Dance”。

### 问题来源

这是一种 强引用 --> 弱引用 --> 强引用 的变换过程。在弄明白为什么要如此大费周章之前，我们首先来看看一般的写法会有什么问题

```
__weak MyViewController *wself = self;
self.completionHandler = ^(NSInteger result) {
    [wself.property removeObserver: wself forKeyPath:@"pathName"];
};

```

这种写法可以避免循环引用，但是我们要考虑这样的问题：

假设 block 被放在子线程中执行，而且执行过程中 self 在主线程被释放了。由于 wself 是一个弱引用，因此会自动变为 nil。而在 KVO 中，这会导致崩溃。

### Strong-Weak Dance

解决以上问题的方法很简单，新增一行代码即可：

```
__weak MyViewController *wself = self;
self.completionHandler = ^(NSInteger result) {
    __strong __typeof(wself) sself = wself; // 强引用一次
    [sself.property removeObserver: sself forKeyPath:@"pathName"];
};

```

这样一来，self 所指向对象的引用计数变成 2，即使主线程中的 self 因为超出作用于而释放，对象的引用计数依然为 1，避免了对象的销毁


1  Q：下面这行代码，将一个弱引用的指针赋值给强引用的指针，可以起到强引用效果么？

```
__strong __typeof(wself) sself = wself;

```
A：会的。引用计数描述的是对象而不是指针。这句话的意思是:
sself 强引用 wself 指向的那个对象 因此对象的引用计数会增加一个

2  Q: block 内部定义了sself，会不会因此强引用了 sself？
A：不会。block 只有截获外部变量时，才会引用它。如果是内部新建一个，则没有任何问题

3 Q: Q：如果在 block 内部没有强引用，而是通过 if 判断，是不是也可以，比如这样写：

```
__weak MyViewController *wself = self;
 wself.completionHandler = ^(NSInteger result) {
     if (wself) { // 只有当 wself 不为 nil 时，才执行以下代码
         [wself.property removeObserver: wself forKeyPath:@"pathName"];
     }
 };
 
```

A：不可以！考虑到多线程执行，也许在判断的时候，self 还没释放，但是执行 self 里面的代码时，就刚好释放了

4 Q: 那按照这个说法，block 内部强引用也没用啊。也许 block 执行以前，self 就释放了。

A：有用！如果在 block 执行以前，self 就释放了，那么 block 的引用计数降为 0，所以自己就会被释放。这样它根本就不会被执行。另外，如果执行一个为 nil 的闭包会导致崩溃。


5 Q: 如果在执行 block 的过程中，block 被释放了怎么办？

A：简单来说，block 还会继续执行，但是它捕获的指针会具有不确定的值，详细内容请参考[这篇文章](https://stackoverflow.com/questions/12272783/what-happens-when-a-block-is-set-to-nil-during-its-execution)


摘自 BestSwifter
 




