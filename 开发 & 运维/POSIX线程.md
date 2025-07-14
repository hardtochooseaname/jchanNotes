# 线程

----

## POSIX线程

AUPE中将的是POSIX线程，也就是POSIX标准中规定的有关线程的接口/API，所以对于不同的UNIX机器，可能有不同的线程实现。



## 线程标识 - TID

进程有进程标识，而且整个系统中不会有重复的PID，不同的PID标识不同的进程。

而TID则不同，线程是从属于进程的，所以TID只在其所属进程的上下文中才有用。所以，不同的进程中也可能分配相同的TID。

#### TID的表示：pthread_t

`pthread_t`是一个结构体，在不同的实现上可能其具体类型会不同，所以在可以移植的程序中不能将其当作普通整数来理解，而应该通过POSIX规定的接口来操作TID。

两个接口函数：

```c
// 比较两个TID
// 相等返回非0，否则返回0
int pthread_equal(pthread_t tid1, pthread_t tid2);

// 获取TID
pthread_t pthread_self(void);
```

### TID的使用

        当主线程安排新的任务进任务队列时，可以通过TID指定该任务由哪个线程来处理。这样可以使线程池中的空闲线程不是随意地从任务队列顶部取下任务，而是只能执行主线程分配给它的任务。

        要做到这一点，可以在任务队列中描述任务的数据结构里放入一个pthread_t变量，然后线程每次从任务队列取任务之前，首先`pthread_self`获取自身TID，再用`pthread_equal`与任务队列中的TID一一比较。



## 线程创建

```c
//  成功返回0，失败直接返回错误码
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                        void *(*start_routine) (void *), void *arg);
```

当创建线程失败时，不会像其他系统调用一样返回-1，然后再通过errno查找到错误原因；而是会直接返回错误码，也不会设置errno。POSIX线程中的所有api都是这样。

#### 几个要注意的点

**1. 新线程开始执行的时机**

在pthread_create调用过程中会创建新线程，但是新线程什么时候开始执行，是pthread_create返回前，还是pthread_create返回后，这是不确定的。

因此，虽然pthread_create会返回新线程的TID，但是最好不要把这个返回的TID通过某种方式传递给新线程使用，因为新线程开始执行的时候pthread_create甚至都还没有返回。

**2. 默认新线程是joinable状态**

这意味着，当线程退出时，线程所占有的资源不会被立即释放，需要其他线程通过pthread_join获取该线程的退出状态后，才会释放。如果想在线程退出时立即释放其资源，则需要将其设置为detached状态

**3. 启动例程的函数类型**

从`pthread_create`函数原型中可以看出，POSIX是这样规定启动例程的类型的：

 `void *(*start_routine) (void *)`

- 形参列表：void *

- 返回值类型：void *



## 线程退出

线程退出有三种方式：

- 启动例程正常return，return的值就是线程的退出码

- `pthread_exit`函数调用

- 被其他线程取消

#### 常规退出

上面的前两种方式都可以看作线程的常规退出方式。

```c
// 空指针指向返回值/退出码
void pthread_exit(void *rval_ptr);

// 第二个参数获取退出码，如果线程被取消则写入PTHREAD_CANCELED
int pthread_join(pthread_t tid, void **tval_ptr);
```

调用pthread_join的线程会一直阻塞，直到等到指定的线程返回，并获取其退出码。如果不关心其退出码，则第二个参数可以是NULL。

通过pthread_join可以实现线程的同步，也可以控制线程资源的释放，获取退出的线程想要传递的信息

**启动例程的返回值类型**

从`pthread_exit`可以看出，线程的退出码是一个空指针，也就是说启动例程的返回值类型也应该是空指针`void *`。这是POSIX线程的规定。

所以启动例程return时，需要把返回值强转为`(void *)`类型。

> 当线程1想要返回整数1时：
> 
> ```c
> return (void*)1;
> ```
> 
> 当其他线程想要获取并使用线程1的返回值时：
> 
> ```c
> int ret;
> pthread_join(tid1, &ret);    // ret中会被写入一个void指针
> printf("thread 1 exit code: %d", (int)ret); // 将void指针强转回int
> ```

当然，该空指针也可以指向一块包含更多信息的数据结构。但是需要保证的是，该数据结构的生命周期需要足够长，至少在线程退出后其他线程pthread_join的时候依旧存在。

#### 线程取消（主动关闭指定线程）

```c
// 成功放回0，否则返回错误码
int pthread_cancel(pthread_t tid);
```

该函数的调用，会使指定线程表现得像是调用了参数为PTHREAD_CANCELED的`pthread_exit`函数。但是线程可以选择忽略取消或者控制如何被取消，这需要线程控制相关的设置。

#### 线程清理处理程序

进程退出时可以通过`atexit`指定退出时调用的清理函数，线程也可以设定这样的函数，叫做*thread cleanup handler*，会在线程退出时被调用。

清理程序是注册到线程的栈中的，所以其执行顺序和注册顺序相反。

**两个调度函数**

```c
// 注册清理函数到栈中
void pthread_cleanup_push(void (*rtn)(void *), void *arg); 
// 传入0时只pop清void理函数不执行，传入非零参数pop并执行
void pthread_cleanup_pop(int execute);
```

当发生以下情况之一时，清理函数被执行

- 调用`pthread_exit`

- 相应取消请求

- 用非零参数调用`pthread_cleanup_pop`

也就是说，**启动例程正常return的时候不会自动调用清理程序**。



**NOTE：**

- 清理程序的返回值类型是void，也就是没有返回值，不要和启动例程的`void*`搞混了

- 线程清理程序需要线程自己注册，不能由主线程统一注册，从`pthread_cleanup_push`没有tid参数就可以看出。

- push和pop两个函数必须要成对出现，否则编译不通过，可以这样
  
  ```c
      pthread_cleanup_push(clientFinished, 0);
      pthread_exit((void*)count);
      pthread_cleanup_pop(0);
  ```
  
  
