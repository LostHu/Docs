###Runloop


线程和**RunLoop**一一对应，Runloop是线程的基础架构部分
> 主线程的runloop默认是启动的，在app工程main函数里
> ```
> int main(int argc, char * argv[]) {
>    @autoreleasepool {
>        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
>    }
>}
> ```
> 对于其他线程的runloop默认是不启动的，如果需要与线程更多交互可以手动配置和启动，如果线程只是执行一个长时间确定的任务则不需要。
> 你只能在一个线程内部获取其Runloop，除了主线程。



一个RunLoop包含若干个Mode，RunLoop 分Mode，主要用来区分**优先级**，比如主线程内置下面2个Mode：
>- **kCFRunLoopDefaultMode**，App默认Mode，通常主线程在这个Mode下运行。
>- **UITrackingRunLoopMode**，界面跟踪Mode，用于ScrollView追踪触摸滑动，保证界面不受其他Mode影响。

每个Mode里包含若干 **Source/Timer/Observer**
>这些统称为Mode item
>Source是事件发起方
>Timer是定时器处理
>Observer是runloop状态观察者
Common

每个RunLoop内部都执行**do-while**事件循环，收到消息处理消息，否则等待。
>RunLoop的核心就是一个mach_msg() ***切换到内核态去执行***，RunLoop调用这个函数去接收消息。
>如果没有取到port消息，内核会将线程置于等待状态。


Mach对象间不能直接调用，只能通过消息传递的方式实现对象间的通信。**消息**是Mach中最基础的概念，消息在两个端口（port）之间传递，这是Mach的IPC（进程间通信）的核心。
RunLoop核心是基于mach port的，其进入休眠时调用的函数是mach_mas().
OSX/iOS的系统架构核心Darwin，包括内核、驱动、Shell等。
Darwin其中在硬件层面有3个组成部分：Mach、BSD、IOKit，共同组成了XNU内核。


####1. AutoreleasePool
App启动后，系统在主线程RunLoop里注册了2个Observer
> - 第一个是Entry，即将进入Loop，回调里会调用```_objc_autoreleasePoolPush()```创建自动释放池，其order = -2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。
> - 第二个监听了2个事件：
> 1. BeforeWaiting（准备进入休眠）调用```_objc_autoreleasePoolPop()```和```_objc_autoreleasePoolPush()```来释放旧池并创建新池；
> 2. Exit（即将退出Loop）调用_objc_autoreleasePoolPop()来释放自动释放池。此order = 2147483647，优先级最低，保证释放发生在其他所有回调之后。

在主线程执行的代码，通常写在诸如事件回调、Timer回调内的，会被RunLoop创建好的AutoreleasePool环绕着，所以不会出现内存泄露，开发者也不必显示创建Pool了。

####2. 事件响应
苹果注册了一个Source1（基于mach port的）用来接收系统事件，回调函数为```__IOHIDEventSystemClineQueueCallback()```。
当一个硬件事件发生后，首先由IOKit.framework生成一个IOHIDEvent事件，随后用**mach port转发给需要的App进程**。随后苹果注册的Source1就会触发回调，并调用```_UIApplicationHandleEventQueue()```进行应用内部的分发。
```_UIApplicationHandleEventQueue()```会把IOHIDEvent处理并包装成UIEvent进行处理或分发。

####3.PerformSelecter
当调用NSObject的```performSelecter:afterDelay:```后，实际上其内部会创建一个Timer并添加到当前线程的RunLoop中。如果当前线程没有RunLoop，则此方法也会失效。

###Runloop实际应用

**AFNetworking**
> 单独创建了一个线程，并启动了一个**RunLoop**
> ```
> + (void)networkRequestThreadEntryPoint:(id)__unused object {
>    @autoreleasepool {
>       [[NSThread currentThread] setName:@"AFNetworking"];
>        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
>        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
>       [runLoop run];
>    }
>}
> ```
> 此处port只是为了让runloop不至于退出，并没有用于实际的发送消息。
> 
> **当需要这个后台线程执行任务时，AFNetworking通过调用```[NSObject performSelector:onThread:]```，将这个任务扔到后台线程的RunLoop中。**