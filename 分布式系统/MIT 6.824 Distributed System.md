# MIT 6.824 Distributed System

----

# Lecture 1: Introduction

## Why we build distributed system?

- parallelism(并行性)：多台系统多个CPU并行工作

- fault tolerance：多台机器做同样的工作，当某台机器出错时，有其他机器可以成功完成该工作

- physic：物理上需要机器分布在不同的地方，例如相同的银行业务需要在多地展开

- security/isolated：可能有问题的/不信任的程序单独运行在一个地方，而可信任的安全的部分运行在另一个地方，实现隔离

其中，最重要的是前两点。

## Distributed System is hard

分布式系统总是很复杂的，如果一个工作放在一台机器上就能够完成，那就放在一台机器上，不要使用分布式系统。例如在一台机器上运行排序算法很简单，但是要放在大量的机器上共同为一个大的数据集排序，就会让工作变得很复杂。So, don't consider distributed system unless necessary.

- concurrency：需要考虑多台机器多个CPU的协同工作

- partial failure：机器多了导致系统出错时难以准确定位和处理（机器出错or系统出错or网络出错）

- performance：建立分布式系统的目的就是增加机器的数目以提高解决问题的速度，而当使用1000台机器的时候如何才能让工作效率提高1000倍，需要良好的设计

## Infrastructure

分布式系统的几个主要构成：

- storage：分布式存储 - GFS

- communication：集群中节点之间的数据交换 - RPC

- computation：大量节点上的并行计算 - MapReduce

        一个好的分布式系统，理想情况下应该向用户提供这样一个接口：隐藏分布式系统的所有细节，让用户以为自己在编写一个在单台主机上运行的程序

一下技术同样重要：

- threads：充分利用多核处理器的优势

- concurrency control：多线程下的并发控制 - locks etc.

## Performance

关于分布式系统的性能，有一个最重要的特性：

- scalability：可拓展性
  
  2xcomputers -> 2xthroughput （通过简单的增加机器数量提高工作量/速度）
  
  <img src="..\images/2024-03-01-14-11-28-image.png" title="" alt="" width="341">

## Fault Tolerance

        当我们只使用一台主机时，机器运行出错的概率是非常低的。但是在分布式系统中，有成千上万太机器，这样就会把很小的概率放大，这时我们就会发现整个分布式系统会频频发生故障。因此需要拥有有效的容错机制来应对这种总是发生的问题。

What does it mean:

- availability：可靠性 - 当分布式系统出现某些错误时，系统能够自行解决这些问题，至少不会影响到程序的正常运行

- recoverability：可恢复性 - 当系统发生了不能忽视的问题时，重要的数据不会丢失，工作进度会被记录，并且在问题解决后能够自动地继续运行

How to do it:

- non-volatile storage：非易失存储 - 当系统断电时数据会保留在存储器中

- replication：一份数据保留多个副本

## Consistancy

使用分布式系统时主要进行两种操作：

- Put(k, v)  更新数据

- Get(k) -> v  获取数据

由于分布式系统中，通常一份数据会有几个副本分别存储在不同的地方，这时就会出现一个问题，当put更新数据到一个副本后，不同副本中的数据就会不一样，那么该如何处理这个问题？

1. strong implement：
   
   - 当用户put数据时，会put数据到所有副本所在的主机上
   
   - 当用户get数据时，会get所有副本的数据，然后取用最新版本
   
   缺点：实际情况中，各个副本通常存储在相距很远的多台主机上，导致多台主机之间的通信开销非常之大。

2. weak implement：
   
   - 用户put和get数据都从距离最近的主机，同时通过其他一些机制来保证应用能正常运行，这就避免了大量的通信开销，这也是许多情况下人们更愿意使用weak implement的原因。

## MapReduce

参考MapReduce.md
