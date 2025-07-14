## POLL vs EPOLL

```c
struct pollfd		client[OPEN_MAX];
...
client[0].fd = listenfd;
client[0].events = POLLRDNORM;
...
while(1){
    nready = poll(client, maxi+1, INFTIM);
}
```

```c
struct pollfd {
	int fd;
	short events;
	short revents;
};
```



```c
typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event
{
  uint32_t events;	/* Epoll events */
  epoll_data_t data;	/* User data variable */
} __EPOLL_PACKED;
```

```c
efd = epoll_create1(0);
...
event.data.fd = listen_fd;
event.events = EPOLLIN | EPOLLET;
epoll_ctl(efd, EPOLL_CTL_ADD, listen_fd, &event);
...
while (1) {
    n = epoll_wait(efd, events, MAXEVENTS, -1);
    for (i = 0; i < n; i++) {
        ...
    }
}
```



## epoll_create

```c
int epoll_create(int size);
int epoll_create1(int flags);
        返回值: 若成功返回一个大于0的值，表示epoll实例；若返回-1表示出错
```

epoll_create() 方法创建了一个 epoll 实例，从 Linux 2.6.8 开始，参数 size 被自动忽略，但是该值仍需要一个大于 0 的整数。这个 epoll 实例被用来调用 epoll_ctl 和 epoll_wait，如果这个 epoll 实例不再需要，比如服务器正常关机，需要调用 close() 方法释放 epoll 实例，这样系统内核可以回收 epoll 实例所分配使用的内核资源。



关于这个参数 size，在一开始的 epoll_create 实现中，是用来告知内核期望监控的文件描述字大小，然后内核使用这部分的信息来初始化内核数据结构，在新的实现中，这个参数不再被需要，因为内核可以动态分配需要的内核数据结构。我们只需要注意，每次将 size 设置成一个大于 0 的整数就可以了。



epoll_create1() 的用法和 epoll_create() 基本一致，如果 epoll_create1() 的输入 flags 为 0，则和 epoll_create() 一样，内核自动忽略。可以增加如 EPOLL_CLOEXEC 的额外选项。

`EPOLL_CLOEXEC` 是 Linux 中 epoll 实例的创建标志之一，它用于设置文件描述符（epoll 实例）的 close-on-exec 标志。

当设置了 `EPOLL_CLOEXEC` 标志时，表示在调用 `exec` 函数时会关闭该 epoll 实例。这样可以确保在执行新程序时，不会继续保持对原有 epoll 实例的引用。这通常用于在多进程环境中避免子进程继承不必要的文件描述符。



## epoll_ctl

```c
 int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
        返回值: 若成功返回0；若返回-1表示出错
```

1.  epfd： 是刚刚调用 epoll_create 创建的 epoll 实例描述字，可以简单理解成是 epoll 句柄

2. op：表示想要执行的操作，它有三个选项可供选择：
   
   - EPOLL_CTL_ADD： 向 epoll 实例注册文件描述符对应的事件；
   
   - EPOLL_CTL_DEL：向 epoll 实例删除文件描述符对应的事件；
   
   - EPOLL_CTL_MOD： 修改文件描述符对应的事件。

3. fd：注册的事件的文件描述符，比如一个监听套接字。

4. event：第四个参数表示的是注册的事件，并且可以在这个结构体里设置用户需要的数据，其中最为常见的是使用联合结构里的 fd 字段，表示事件所对应的文件描述符。



#### 几种常用事件类型

- EPOLLIN：表示对应的文件描述字可以读；

- EPOLLOUT：表示对应的文件描述字可以写；

- EPOLLHUP：表示对应的文件描述字被挂起（），对端正常关闭连接或者发生错误导致连接关闭；

- EPOLLERR：当一个文件描述符上发生错误时，就会触发 `EPOLLERR` 事件。这种错误包括连接出现错误、socket 发生错误等情况；

- EPOLLET：设置为 edge-triggered，默认为 level-triggered；



#### edge-triggered VS level-triggered

![Edge Triggering And Level Triggering GeeksforGeeks, 43% OFF](https://bob.cs.sonoma.edu/IntroCompOrg-x64/book173x.png)



## epoll_wait

```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
  返回值: 成功返回的是一个大于0的数，表示事件的个数；返回0表示的是超时时间到；若出错返回-1.
```


