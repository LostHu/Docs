### iOS知识点回顾

1. @property 后面可以有哪些修饰符？

> - **atomic**(**默认**) 原子性，线程安全，防止在写未完成的时候被其他线程读取，自动生成**自旋锁**代码，当上一个线程未执行完下一个线程会一直等待。会消耗大量资源，效率低。
> - **nonatomic** 非原子性，非线程安全，放弃安全，提高性能
> - **assign** 可修饰**基本数据类型**和**OC对象**，不会增加引用计数，如果是指针被释放后，仍指向对应内存，必须手动置**nil**，否则会产生野指针。
> - **weak** 弱引用，只能修饰**OC对象**，如果指向的对象被释放了，会自动置为**nil**，比较安全。
> - **strong**和**retain**都修饰**OC对象**，强引用，**setter**方法会先将旧对象release掉，再将新值赋给该对象并进行一次**retain**操作，都会增加引用计数。**strong**一般用于**ARC**，**retain**用于**MRC**。
> - **strong**和**copy** 
>> - **copy** 一般修饰不可变对象，如果修饰可变对象，则*调用**可变方法**会报错*；在**setter**方法中做如下判断（如果较多copy，需要考量性能）：
> 如果赋值来源是不可变对象时和**strong**一样进行浅拷贝，引用计数+1。
> 如果赋值来源是可变对象则进行深拷贝，不会增加引用计数。
>> - **strong** 修饰不可变对象
> 如果赋值来源是不可变对象没问题
> 如果赋值来源是可变对象会造成**破坏封装性**
>> - **strong** 修饰可变对象
> 如果赋值来源是不可变对象会**报错**
> 如果赋值来源是可变对象没问题

> - **setter**和**getter** 替换成自定义函数
> - **__unsafe_unretained** 与__weak一样，对象即使被销毁指针也不会置空，一般用于与C语言交互

2. @property 的本质是什么？ @property 的作用？
> - @property = ivar + setter + getter (实例变量+存取方法)
> 完成属性定义后，编译器会**自动编写访问这些属性所需的方法**，此过程叫做“**自动合成**”，这个过程由编译器在编译期执行。
> - @property的**作用**是是编译器会**自动添加*getter*和*setter*方法**。

3. **代理**一般如何修饰？**Block**如何修饰？
> 代理一般使用**weak**修饰，如果使用strong会造成循环引用
> Block一般使用**copy**修饰，只有copy后的block才会在堆中。

4. @protocol和category中如何使用@property

> - 都只会生成getter和setter方法声明。
> - 如果要给category增加属性需要使用运行时函数：
> 1. objc_setAssociatedObject
> 2. objc_getAssociatedObject

5. 如何让自己的类用copy修饰符？如何重写带copy的关键字的setter？
> 需要实现**NSCopying**协议，如果需要区分可变和不可变版本，则还需要实现**NSMutableCopying**协议

6. **@synthesize** 和 **@dynamic** 分别有什么作用？
> - 如果2个都没写，则默认@synthesize var = _var;
> - @**synthesize**如果没有手动实现**setter和getter**方法，则编译器**自动为你加上**这2个方法。
> - @**dynamic**告诉编译器**getter和setter**方法**由用户自己生成**，如果没提供这2个方法，编译通过，但运行时使用到这2个方法则会崩溃。

7. **ARC**下，不显式指定任何属性关键字，默认关键字？

> - 基本数据类型：atomic readwrite assign
> - OC对象：          atomic readwrite strong

8. **autorelease**对象什么时候释放？
> - 手动干预，指定autoreleasepool的大括号结束时释放
> - 自动释放，对象出了作用域之后，会被添加到最近一次创建的自动释放池。**系统检测到事件并且启动后，即会创建自动释放池**

9. KVO实现原理？
> - 当你观察一个对象时候，一个**新类**被**动态创建**，新类**继承自被观察对象**的类，并**重写了setter**方法，并将此对象的isa指针指向新类(**isa-swizzling**)，此对象就变成了新类的实例。
> ```
> - (void)setA:(NSString*)a
> {
>	[self willChangeValueForKey:@"a"];
>	[super setValue:a forKey:@"a"];
>	[self didChangeValueForKey:@"a"];
> }
> ```

10. 如何高效实现对UIImageView加圆角？
> 工作线程利用CoroGraphic API 生成一个offscreen UIImage，切换到主线程赋值给ImageView。需要考虑绘制缓存问题。

11. drawRect？
> 利用**CPU**生成offscreen bitmap，问题是复杂+内存开销大，减轻GPU绘制压力，一般需要平衡CPU和GPU压力。

12. 常用的设计模式？
> - 单例模式
> - 观察者模式
> - 响应者模式
> - 组合模式
> - 代理模式


13. APNs流程？

**Apple Push Notification Server**：设备和APNs服务器之间的通讯是基于SSL协议的TCP长连接。

> 1. 开发者注册配置APNs消息推送 （申请证书配置，2种，开发和发布环境）
> 2. APP启动时注册APNs服务器，参数为设备序列号
> 3. APNs经过处理后将产生的device_token返回给设备
> 4. APP使用device_token向自己的应用服务器注册
> 5. 应用服务器将推送内容+device_token打包后发给APNs
> 6. APNs推送至设备


14. APP生命周期？

>- **Not running**
>- **Inactive**：运行在前台+没有接收事件。状态切换，会短暂停留。长期停留（锁屏、来电、短信等）
>- **Active**：运行在前台+接收事件
>- **Background**：运行在后台+执行代码
>- **Suspended**：运行在后台+不执行代码

15. 链式语法实现原理？
> - 调用函数方法 ```[object a];``` 
> -  如果需要```[[object a] b];```继续调用下一个b方法，则需要a方法**返回object对象本身**。
> - 如果需要实现```object.a();```，则需要使用block，一般调用block的语法是此样式。

16. ARC ```dealloc()``` 方法？
--
> - 要释放CoreFoundation对象，这个不归ARC管理，需要调用```CFRetain/CFRelease```。
> - 取消```Notification```订阅。

17. 消息转发全流程？
--
> - 总结流程
>  - - **首先看是否动态添加了实现**  
>  - - **尝试转发其他某个对象处理**
>  - - **尝试打包方法签名**
>  - - **尝试自定义分发（可多对象）处理**
- 首先会走``` resolveInstanceMethod:(SEL)sel 或 resolveClassMethod:(SEL)sel ```，看实例对象或者类是否在运行时添加了对应的```sel```实现
- 如果没有，走``` (id)forwardingTargetForSelector:(SEL)sel```，如果返回不为空，则将消息交给返回对象去处理这个```sel```。只能交给单个对象。
- 如上一部返回nil，交给``` -(NSMethodSignature*)methodSignatureForSelector:(SEL)sel ```处理生成这个```sel```的签名，包含这个sel的所有信息。
- 如果返回签名成功，则```-(void)forwardInvocation:(NSInvocation*)incovation```，则可以调用```NSInvocation```来做最后的处理。可以转发给多个对象。