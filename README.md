
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

OBJECTIVE-C
当然第一步要包含头文件
> `#import <pthread.h>`

然后创建线程，并执行任务</br>
`` - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {``</br>
   `` pthread_t thread;``</br>
   ``//创建一个线程并自动执行``<br>
   `` pthread_create(&thread, NULL, start, NULL);``</br>
  ``} ``</br>
  ``void *start(void *data) {``</br>
  ``NSLog(@"%@", [NSThread currentThread]);``</br>
  ``return NULL;``</br>
``}``</br>
打印输出：</br>
``2015-07-27 23:57:21.689 testThread[10616:2644653] <NSThread: 0x7fbb48d33690>{number = 2, name = (null)}``</br>
看代码就会发现他需要 c语言函数，这是比较蛋疼的，更蛋疼的是你需要手动处理线程的各个状态的转换即管理生命周期，比如，这段代码虽然创建了一个线程，但并没有销毁。
