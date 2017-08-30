#### Tencent Mars

腾讯的微信团队在2016年12月份开源的跨平台业务的终端基础组件

![](/img/post-img-mars.png)

- comm：可以独立使用的公共库，包括 socket、线程、消息队列、协程等；
- xlog：高可靠性高性能的运行期日志组件；
- SDT： 网络诊断组件；
- STN： 信令分发网络模块，也是 Mars 最主要的部分。

#### Android marsSampleChat

##### Queue 归纳

```
public interface Queue<E> extends Collection<E>
```

除了Collection的操作外，Queue还提供：

* boolean add(E e)
* boolean offer(E e)
* E remove();
* E poll();

Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列，其中阻塞队列的经典例子是BlockingQueue，非阻塞队列的经典例子是ConcurrentLinkedQueue。

* BlockingQueue中主要用到put和take方法，put方法在队列满的时候回阻塞直到有队列成员被消费，take方法在队列空的时候会阻塞，知道队列成员被放进来
* ConcurrentLinkedQueue是Queue的一个安全实现，元素按照FIFO原则进行排序，LinkedBlockingQueue是一个线程安全的阻塞队列，实现了BlockingQueue接口

##### ServerConnection 归纳

