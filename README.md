
### 关于iOS的多线程
在iOS中目前有4套多线程方案，他们分别是：</br>
> Pthreads</br>
> NSThread</br>
> GCD</br>
> NSOperation & NSOPerationQueue</br>

下面就是各种线程的使用方法

## Pthreads
其实这个方案不用说的，只是拿来充个数，为了让大家了解一下就好了。百度百科里是这么说的：

>POSIX线程（POSIX threads），简称Pthreads，是线程的POSIX标准。该标准定义了创建和操纵线程的一整套API。在类Unix操作系统（Unix、Linux、Mac OS X等）中，都使用Pthreads作为操作系统的线程。

简单地说，这是一套在很多操作系统上都通用的多线程API，所以移植性很强（然并卵），当然在 iOS 中也是可以的。不过这是基于 c语言 的框架，使用起来这酸爽！感受一下：

OBJECTIVE-C</br>
当然第一步要包含头文件</br>
> `#import <pthread.h>`

然后创建线程，并执行任务</br>
 <pre>- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
   pthread_t thread;
   //创建一个线程并自动执行`
    pthread_create(&thread, NULL, start, NULL);
  } 
  void *start(void *data) {
  NSLog(@"%@", [NSThread currentThread]);
  return NULL;
}
打印输出：
2015-07-27 23:57:21.689 testThread[10616:2644653] <NSThread: 0x7fbb48d33690>{number = 2, name = (null)}
</pre>
看代码就会发现他需要 c语言函数，这是比较蛋疼的，更蛋疼的是你需要手动处理线程的各个状态的转换即管理生命周期，比如，这段代码虽然创建了一个线程，但并没有销毁。

那么，Pthreads 方案的多线程我就介绍这么多，毕竟做 iOS 开发几乎不可能用到。但是如果你感兴趣的话，或者说想要自己实现一套多线程方案，从底层开始定制，那么可以去搜一下相关资料。

##NSThread
这套方案是经过苹果封装后的，并且完全面向对象的。所以你可以直接操控线程对象，非常直观和方便。但是，它的生命周期还是需要我们手动管理，所以这套方案也是偶尔用用，比如`[NSThread currentThread]`，它可以获取当前线程类，你就可以知道当前线程的各种属性，用于调试十分方便。下面来看看它的一些用法。

###创建并启动
* 先创建线程类，再启用</br>
  OBJECTIVE-C
   <pre> // 创建
  NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run:) object:nil];
  // 启动
  [thread start];
  </pre>
* 创建并自动启用
  <pre>
  [NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:nil];
  </pre>
* 使用NSObject的方法创建并自动启用<br>
  OBJECTIVE-C
  <pre>[self performSelectorInBackground:@selector(run:) withObject:nil];</pre>
  
###其他方法
除了创建启动外，NSThread 还以很多方法，下面我列举一些常见的方法，当然我列举的并不完整，更多方法大家可以去类的定义里去看。

OBJECTIVE-C  
<pre>
//取消线程
- (void)cancel;
//启动线程
- (void)start;
//判断某个线程的状态的属性
@property (readonly, getter=isExecuting) BOOL executing;
@property (readonly, getter=isFinished) BOOL finished;
@property (readonly, getter=isCancelled) BOOL cancelled;
//设置和获取线程名字
-(void)setName:(NSString *)n;
-(NSString *)name;
//获取当前线程信息
+ (NSThread *)currentThread;
//获取主线程信息
+ (NSThread *)mainThread;
//使当前线程暂停一段时间，或者暂停到某个时刻
+ (void)sleepForTimeInterval:(NSTimeInterval)time;
+ (void)sleepUntilDate:(NSDate *)date;</pre>

其实，NSThread 用起来也挺简单的，因为它就那几种方法。同时，我们也只有在一些非常简单的场景才会用 NSThread, 毕竟它还不够智能，不能优雅地处理多线程中的其他高级概念。所以接下来要说的内容才是重点。