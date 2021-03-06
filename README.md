# GCDTimer-demo
A timer implementation that uses dispatch_source_t(GCD)，API is just like NSTimer.

####  为什么要封装GCDTimer
 NSTimer一些众所周知的不便
- 在常用场景下对self强引用，引起的循环引用
- 由于Runloop在任务繁重时所引起的timer时间不准


针对第一个问题解决方案有很多：

- 使用第三方对象T作为Timer的target，然后弱引用T，通过T消息转发到timer事件上，来打破循环。
- 使用关联对象，关联第三方对象，并将对象绑定一个C语言函数，然后通过函数调用当前的timer事件。
- 使用block类型的NSTimer事件，并且使用__weak,__strong打破循环,如：

```objc
 __weak XXClass *weakSelf = self;
    timer = [NSTimer xx_scheduledTimerWithTimeInterval:.5 block:^{
                XXClass *strongSelf = weakSelf;
                [strongSelf doSomething];
            }repeats:YES];
```
当然，上面方案有好有坏，欢迎讨论，我在项目中是使用GCD源定时器作为一般场景的定时器来取代NSTimer使用。本工程是对GCD信号源定时器的封装。当然，郭曜源大神在他开源的YYKit当中的封装了一个更加线程安全的版本[YYTimer](https://github.com/ibireme/YYKit/tree/3869686e0e560db0b27a7140188fad771e271508/YYKit/Utility）



另外，官方并非没有注意到这个事情，在iOS 10以后，NSTimer的API中出现了下面这几个方法，看一下注释就很清楚，已经处理了造成循环引用的问题。所以推荐使用这一组方法。

```objective-c

/// Creates and returns a new NSTimer object initialized with the specified block object. This timer needs to be scheduled on a run loop (via -[NSRunLoop addTimer:]) before it will fire.
/// - parameter:  timeInterval  The number of seconds between firings of the timer. If seconds is less than or equal to 0.0, this method chooses the nonnegative value of 0.1 milliseconds instead
/// - parameter:  repeats  If YES, the timer will repeatedly reschedule itself until invalidated. If NO, the timer will be invalidated after it fires.
/// - parameter:  block  The execution body of the timer; the timer itself is passed as the parameter to this block when executed to aid in avoiding cyclical references
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

/// Creates and returns a new NSTimer object initialized with the specified block object and schedules it on the current run loop in the default mode.
/// - parameter:  ti    The number of seconds between firings of the timer. If seconds is less than or equal to 0.0, this method chooses the nonnegative value of 0.1 milliseconds instead
/// - parameter:  repeats  If YES, the timer will repeatedly reschedule itself until invalidated. If NO, the timer will be invalidated after it fires.
/// - parameter:  block  The execution body of the timer; the timer itself is passed as the parameter to this block when executed to aid in avoiding cyclical references
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

/// Initializes a new NSTimer object using the block as the main body of execution for the timer. This timer needs to be scheduled on a run loop (via -[NSRunLoop addTimer:]) before it will fire.
/// - parameter:  fireDate   The time at which the timer should first fire.
/// - parameter:  interval  The number of seconds between firings of the timer. If seconds is less than or equal to 0.0, this method chooses the nonnegative value of 0.1 milliseconds instead
/// - parameter:  repeats  If YES, the timer will repeatedly reschedule itself until invalidated. If NO, the timer will be invalidated after it fires.
/// - parameter:  block  The execution body of the timer; the timer itself is passed as the parameter to this block when executed to aid in avoiding cyclical references
- (instancetype)initWithFireDate:(NSDate *)date interval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

```





#### 使用

将demo中的`GCDTimer`类拖到你的工程中就可以使用。
GCDTimer的接口是模仿NSTimer的设计的，分别提供了2个类方法1个实例方法。

```objc
GCDTimer *timer1 = [GCDTimer scheduledTimerWithTimerInterval:1 repeats:YES block:^(dispatch_source_t timer) {
        NSLog(@"Hello GCDTimer");
}];
    
//可以创建自己想要的指定队列传入queue初始化定时器，默认在全局并发队列中运行定时器
dispatch_queue_t queue = dispatch_queue_create("timerQueue", DISPATCH_QUEUE_CONCURRENT);
GCDTimer *timer2 = [GCDTimer timerWithTimeInterval:1 leeway:0 queue:queue repeats:YES block:^(dispatch_source_t timer) {
        NSLog(@"Hello GCDTimer");
}];
GCDTimer *timer13 = [[GCDTimer alloc]initWithTimerInterval:1 leeway:0 queue:nil repeats:YES block:^(dispatch_source_t timer) {
        NSLog(@"Hello GCDTimer");
}];
    
```
Timer的生命周期：
```objc
//启动timer
- (void)resume;
//暂停
- (void)pause;
//取消调度源
- (void)cancel;
```

#### 最后
欢迎在issue区一起讨论。
