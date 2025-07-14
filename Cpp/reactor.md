[高性能网络框架：Reactor 和 Proactor-云社区-华为云](https://bbs.huaweicloud.com/blogs/266248?utm_source=zhihu&utm_medium=bbs-ex&utm_campaign=other&utm_content=content)

## 单 Reactor 单进程 / 线程

![](https://img-blog.csdnimg.cn/img_convert/3b2723e04cf760ee137d87bf8e99d471.png)

可以看到进程里有 **Reactor、Acceptor、Handler** 这三个对象：

- Reactor 对象的作用是监听和分发事件 -> 裸epoll中的epoll_wait和for循环
- Acceptor 对象的作用是获取连接 -> 负责建立连接相关事宜
- Handler 对象的作用是处理业务 -> 一个handler和对应一条连接，连接上有事件就绪则分发给对应handler处理

但是，这种方案存在 2 个缺点：

- 第一个缺点，因为只有一个进程，**无法充分利用 多核 CPU 的性能**；
- 第二个缺点，Handler 对象在业务处理时，整个进程是无法处理其他连接的事件的，**如果业务处理耗时比较长，那么就造成响应的延迟**；

#### 单 Reactor 多线程 / 多进程

![](https://img-blog.csdnimg.cn/img_convert/ed58e04908567b94b84df8bff2d5637c.png)

为了解决单reactor单线程带来的第二个缺点，把可能因为耗时过长而影响整个应用并发效率的业务处理，单独分发给另外的线程进行处理。因此，此处使用线程池是为了专门负责任务处理的。

另外，「单 Reactor」的模式还有个问题，**因为一个 Reactor 对象承担所有事件的监听和响应，而且只在主线程中运行，在面对瞬间高并发的场景时，容易成为性能的瓶颈的地方**。

#### 

#### 多 Reactor 多进程 / 线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426221552103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODI3Njc0,size_16,color_FFFFFF,t_70)

这里使用多线程，是用来处理所有已建立的连接上数据的接收、处理、发送。

每个子线程中有一个subreactor，负责由主线程分发的部分连接
