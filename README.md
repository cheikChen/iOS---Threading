
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
	+ (void)sleepUntilDate:(NSDate *)date;

其实，NSThread 用起来也挺简单的，因为它就那几种方法。同时，我们也只有在一些非常简单的场景才会用 NSThread, 毕竟它还不够智能，不能优雅地处理多线程中的其他高级概念。所以接下来要说的内容才是重点。

##GCD
`Grand Central Dispatch`，听名字就霸气。它是苹果为多核的并行运算提出的解决方案，所以会自动合理地利用更多的CPU内核（比如双核、四核），最重要的是它会自动管理线程的生命周期（创建线程、调度任务、销毁线程），完全不需要我们管理，我们只需要告诉干什么就行。同时它使用的也是 c语言，不过由于使用了 Block（Swift里叫做闭包），使得使用起来更加方便，而且灵活。所以基本上大家都使用` GCD` 这套方案，老少咸宜，实在是居家旅行、杀人灭口，必备良药。不好意思，有点中二，咱们继续。

###任务和队列
在 `GCD` 中，加入了两个非常重要的概念： **任务** 和 **队列**。

**任务**：即操作，你想要干什么，说白了就是一段代码，在 GCD 中就是一个 Block，所以添加任务十分方便。任务有两种执行方式： 同步执行 和 异步执行，他们之间的区别是 是否会创建新的线程。

**同步执行**：只要是同步执行的任务，都会在当前线程执行，不会另开线程。

**异步执行**：只要是异步执行的任务，都会另开线程，在别的线程执行。

>同步（sync） 和 异步（async） 的主要区别在于会不会阻塞当前线程，直到 Block 中的任务执行完毕！
如果是 同步（sync） 操作，它会阻塞当前线程并等待 Block 中的任务执行完毕，然后当前线程才会继续往下运行。
如果是 异步（async）操作，当前线程会直接往下执行，它不会阻塞当前线程。

**队列**：用于存放任务。一共有两种队列， **串行队列** 和 **并行队列**。

 **串行队列**：放到串行队列的任务，GCD 会 FIFO（先进先出） 地取出来一个，执行一个，然后取下一个，这样一个一个的执行。
 
 **并行队列**：放到并行队列的任务，GCD 也会 FIFO的取出来，但不同的是，它取出来一个就会放到别的线程，然后再取出来一个又放到另一个的线程。这样由于取的动作很快，忽略不计，看起来，所有的任务都是一起执行的。不过需要注意，GCD 会根据系统资源控制并行的数量，所以如果任务很多，它并不会让所有任务同时执行。
 
 请看下表：
 
|			|	同步执行	|	异步执行|
|---------|------------|---------|
|串行队列	|当前线程，一个一个执行	|其他线程，一个一个执行|
|并行队列	|当前线程，一个一个执行	|开很多线程，一起执行|

###创建队列
* **主队列**：这是一个特殊的 串行队列。什么是主队列，大家都知道吧，它用于刷新 UI，任何需要刷新 UI 的工作都要在主队列执行，所以一般耗时的任务都要放到别的线程执行。

`dispatch_queue_t queue = ispatch_get_main_queue();`

* **自己创建的队列**：自己可以创建 串行队列, 也可以创建 并行队列。看下面的代码，它有**两个参数**，其中第一个参数是标识符，用于 DEBUG 的时候标识唯一的队列，可以为空。大家可以看xcode的文档查看参数意义。第二个参数用来表示创建的队列是串行的还是并行的，传入 `DISPATCH_QUEUE_SERIAL` 或 `NULL` 表示创建串行队列。传入 `DISPATCH_QUEUE_CONCURRENT `表示创建并行队列。

<pre>
//串行队列
  dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", NULL);
  dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_SERIAL);
  //并行队列
  dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_CONCURRENT);
</pre>

* **全局并行队列**：只要是并行任务一般都加入到这个队列。这是系统提供的一个并发队列。

`dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);`



###创建任务
* **同步任务**：会阻塞当前线程 (SYNC)
<pre>
dispatch_sync(<#queue#>, ^{
      //code here
      NSLog(@"%@", [NSThread currentThread]);
  });
</pre>
* **异步任务**：不会阻塞当前线程 (ASYNC)
<pre>
  dispatch_async(<#queue#>, ^{
      //code here
      NSLog(@"%@", [NSThread currentThread]);
  });
</pre>
为了更好的理解同步和异步，和各种队列的使用，下面看两个示例：</br>

>###示例一：
以下代码在主线程调用，结果是什么？</br>
<pre>
NSLog("之前 - %@", NSThread.currentThread())
dispatch_sync(dispatch_get_main_queue(), { () -> Void in 
        NSLog("sync - %@", NSThread.currentThread())
})
NSLog("之后 - %@", NSThread.currentThread())
</pre>
**答案：**</br>
只会打印第一句：之前 `- <NSThread: 0x7fb3a9e16470>{number = 1, name = main} `，然后主线程就卡死了，你可以在界面上放一个按钮，你就会发现点不了了。</br>
**解释：**</br>
同步任务会阻塞当前线程，然后把 Block 中的任务放到指定的队列中执行，只有等到 Block 中的任务完成后才会让当前线程继续往下运行。
那么这里的步骤就是：打印完第一句后，`dispatch_sync` 立即阻塞当前的主线程，然后把 Block 中的任务放到 `main_queue` 中，可是 `main_queue` 中的任务会被取出来放到主线程中执行，但主线程这个时候已经被阻塞了，所以 Block 中的任务就不能完成，它不完成，`dispatch_sync` 就会一直阻塞主线程，这就是死锁现象。导致主线程一直卡死。

>###示例二：
以下代码会产生什么结果？
<pre>
let queue = dispatch_queue_create("myQueue", DISPATCH_QUEUE_SERIAL)
   NSLog("之前 - %@", NSThread.currentThread())
    dispatch_async(queue, { () -> Void in
        NSLog("sync之前 - %@", NSThread.currentThread())
        dispatch_sync(queue, { () -> Void in
             NSLog("sync - %@", NSThread.currentThread())
        })
        NSLog("sync之后 - %@", NSThread.currentThread())
   })
  NSLog("之后 - %@", NSThread.currentThread())
</pre>
**答案：**</br>
2015-07-30 02:06:51.058 test[33329:8793087] 之前 - <NSThread: 0x7fe32050dbb0>{number = 1, name = main}
2015-07-30 02:06:51.059 test[33329:8793356] sync之前 - <NSThread: 0x7fe32062e9f0>{number = 2, name = (null)}
2015-07-30 02:06:51.059 test[33329:8793087] 之后 - <NSThread: 0x7fe32050dbb0>{number = 1, name = main}
很明显 `sync - %@ `和 `sync之后 - %@` 没有被打印出来！这是为什么呢？我们再来分析一下：</br>
**分析：**</br>
我们按执行顺序一步步来哦：
使用 `DISPATCH_QUEUE_SERIAL` 这个参数，创建了一个 `串行队列`。
打印出 `之前 - %@` 这句。
`dispatch_async` 异步执行，所以当前线程不会被阻塞，于是有了两条线程，一条当前线程继续往下打印出 `之后 - %@`这句, 另一台执行 Block 中的内容打印 `sync之前 - %@` 这句。因为这两条是并行的，所以打印的先后顺序无所谓。</br>
注意，高潮来了。现在的情况和上一个例子一样了。`dispatch_sync`同步执行，于是它所在的线程会被阻塞，一直等到 sync 里的任务执行完才会继续往下。于是 sync 就高兴的把自己 Block 中的任务放到 queue 中，可谁想 queue 是一个串行队列，一次执行一个任务，所以 sync 的 Block 必须等到前一个任务执行完毕，可万万没想到的是 queue 正在执行的任务就是被 sync 阻塞了的那个。于是又发生了死锁。所以 sync 所在的线程被卡死了。剩下的两句代码自然不会打印。

###队列组
队列组可以将很多队列添加到一个组里，这样做的好处是，当这个组里所有的任务都执行完了，队列组会通过一个方法通知我们。下面是使用方法，这是一个很实用的功能。
<pre>
//1.创建队列组
dispatch_group_t group = dispatch_group_create();
//2.创建队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
//3.多次使用队列组的方法执行任务, 只有异步方法
//3.1.执行3次循环
dispatch_group_async(group, queue, ^{
    for (NSInteger i = 0; i < 3; i++) {
        NSLog(@"group-01 - %@", [NSThread currentThread]);
    }
});
//3.2.主队列执行8次循环
dispatch_group_async(group, dispatch_get_main_queue(), ^{
    for (NSInteger i = 0; i < 8; i++) {
        NSLog(@"group-02 - %@", [NSThread currentThread]);
    }
});
//3.3.执行5次循环
dispatch_group_async(group, queue, ^{
    for (NSInteger i = 0; i < 5; i++) {
        NSLog(@"group-03 - %@", [NSThread currentThread]);
    }
});
//4.都完成后会自动通知
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"完成 - %@", [NSThread currentThread]);
});
</pre>
打印结果
>2015-07-28 03:40:34.277 test[12540:3319271] group-03 - <NSThread: 0x7f9772536f00>{number = 3, name = (null)}</br>
2015-07-28 03:40:34.277 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.277 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.277 test[12540:3319271] group-03 - <NSThread: 0x7f9772536f00>{number = 3, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.278 test[12540:3319271] group-03 - <NSThread: 0x7f9772536f00>{number = 3, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319271] group-03 - <NSThread: 0x7f9772536f00>{number = 3, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.277 test[12540:3319273] group-01 - <NSThread: 0x7f977272e8d0>{number = 2, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319271] group-03 - <NSThread: 0x7f9772536f00>{number = 3, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.278 test[12540:3319273] group-01 - <NSThread: 0x7f977272e8d0>{number = 2, name = (null)}</br>
2015-07-28 03:40:34.278 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.278 test[12540:3319273] group-01 - <NSThread: 0x7f977272e8d0>{number = 2, name = (null)}</br>
2015-07-28 03:40:34.279 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.279 test[12540:3319146] group-02 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>
2015-07-28 03:40:34.279 test[12540:3319146] 完成 - <NSThread: 0x7f977240ba60>{number = 1, name = main}</br>

这些就是 GCD 的基本功能，但是它的能力远不止这些，等讲完 NSOperation 后，我们再来看看它的一些其他方面用途。而且，只要你想象力够丰富，你可以组合出更好的用法。

>更新：关于GCD，还有两个需要说的：
`func dispatch_barrier_async(_ queue: dispatch_queue_t, _ block: dispatch_block_t)`:
这个方法重点是你传入的 `queue`，当你传入的 `queue` 是通过 `DISPATCH_QUEUE_CONCURRENT` 参数自己创建的 `queue `时，这个方法会阻塞这个 `queue`（注意是阻塞 `queue` ，而不是阻塞当前线程），一直等到这个 `queue` 中排在它前面的任务都执行完成后才会开始执行自己，自己执行完毕后，再会取消阻塞，使这个 `queue` 中排在它后面的任务继续执行。
如果你传入的是其他的 `queue`, 那么它就和 `dispatch_async` 一样了。
`func dispatch_barrier_sync(_ queue: dispatch_queue_t, _ block: dispatch_block_t)`:
这个方法的使用和上一个一样，传入 自定义的并发队列`（DISPATCH_QUEUE_CONCURRENT）`，它和上一个方法一样的阻塞 `queue`，不同的是 这个方法还会 阻塞当前线程。
如果你传入的是其他的 `queue`, 那么它就和 `dispatch_sync` 一样了。