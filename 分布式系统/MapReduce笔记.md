# MapReduce

--------

## 1. What is it?

        这是一个谷歌内部开发的用于分布式系统的并行计算框架。

#### 产生的背景

在开发此框架之前，谷歌的许多日常业务中涉及到大量数据的处理计算：

- 爬取的大量网页

- web请求的日志数据

- 浏览器查询记录

        为了加速对这些大量的数据的处理，谷歌会将这些计算分散到集群中的大量主机上并行计算。而谷歌程序员通常会为不同的业务需求编写不同的代码，久而久之，发现这些不同的代码中，有很多相似和重复的部分，这些部分都是**并行计算的细节、错误处理、数据的分布、负载均衡**等相关的。

        同时，对于数据处理的逻辑也杂糅到前者中，使得原本简单的计算逻辑（词频统计、排序）也变得复杂。所以，后面谷歌专门开发出mapreduce并行计算框架来封装这一部分重复的工作，并且让用户可以专注于数据处理本身。

        数据处理的逻辑放在用户自定义函数Map和Reduce中。这是最核心的两个函数。

        MapReduce框架最核心的一个目标就是：让用户编程变得更简单方便

## 2. MapReduce原理的简单概述

#### 键值对数据

MapReduce中，所有数据都是以键值对的形式出现，输入的是键值对，中间计算结果是键值对，输出也是键值对。

键值对（Key-Value Pair）是一种常用的数据结构，使用键（Key）可以快速找到对应的值（Value）。在MapReduce模型中，使用键值对的形式进行输入输出有以下几个主要原因：

1. 通用性：键值对的形式可以适应各种类型的数据和各种类型的计算任务。例如，键可以是文档的ID，值可以是文档的内容；键可以是单词，值可以是该单词出现的次数等。

2. 并行性：在Map阶段，不同的键值对可以被分配到不同的Map任务进行并行处理。在Reduce阶段，所有具有相同键的键值对会被分配到同一个Reduce任务进行处理。这种以键为基础的分配策略使得MapReduce能够有效地在多个计算节点上进行并行计算。

3. 易于处理和优化：使用键值对的形式，可以方便地进行排序、分组等操作。例如，在Shuffle阶段，MapReduce框架会自动地对所有的键进行排序，并将相同的键分组在一起，这为Reduce阶段的处理提供了便利。

4. 易于理解和编程：使用键值对的形式，开发者只需要关注如何实现Map函数和Reduce函数，而不需要考虑数据的分布、并行计算、容错等问题，这些都由MapReduce框架自动处理。这使得MapReduce模型易于理解和编程

#### Map & Reduce：

在使用MapReduce框架的时候，用户的计算逻辑放在两个核心函数中：map和reduce。这两个函数也是需要用户编写的两个主要函数，这两个函数就分别对应两个计算阶段。

![](..\images\2024-02-29-16-04-45-image.png)

**Map阶段:**

map接受一个键值对作为输入，对该数据处理后，输出一个中间键值对的列表。

**Reduce阶段:**

MapReduce会将众多map的输出汇总，将相同key的value集合起来，得到不同的key对应的value列表，再将这样一个(key, list(value))键值对传入reduce函数。而reduce函数，顾名思义，需要对key的众多value进行处理，使得最后一个key只剩下一个或者零个value。

#### 一个简单的例子：词频统计

<img title="" src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-03-26-00-21-56-image.png" alt="" width="436">

**Emit:**

"emit"在MapReduce中的含义，与传统编程中的"return"有一些不同。"return"通常是指一个函数完成计算后，将结果返回给调用者。而"emit"则是在MapReduce的Map和Reduce函数中，将中间结果（键值对）输出，以便后续处理。

具体来说，Map函数和Reduce函数在处理数据时，会产生一些中间结果，这些结果需要以键值对的形式，被传递到下一阶段（例如，从Map阶段传递到Reduce阶段）。这个传递过程，就是所谓的"emit"操作。"emit"的结果并不直接返回给调用者，而是被MapReduce框架接收并进行后续处理。

**MapReduce规范对象**

用户在使用MapReduce框架时，除了需要编写map和reduce函数，还需要在代码中创建一个MapReduce规范对象用于存储配置信息：

- 输入文件名（位置、URL）

- 输出文件名（位置）

- 一些其他参数（并行度、是否需要key排序等）

用户在创建了这个MapReduce规范对象后，再在调用**MapReduce函数**时将该对象作为参数传给MapReduce函数，这样来控制MapReduce的分布式并行计算的具体细节。

## 3. 具体实现

针对不同的环境，同一个MapReduce框架需要不同的具体实现来确保其高效性，而谷歌的MapReduce实现所依靠的环境（2004年），是一个千兆以太网中的数千台commodity PCs所组成的集群。

  ![](..\images\2024-02-29-16-06-24-image.png)

#### 具体工作流程：

1. 从**用户程序**开始运行，根据用户的配置信息，调用MapReduce函数，然后MapReduce框架开始接管计算任务。首先会找到输入文件，并将输入文件划分成M个16~64MB的片段（split），然后启动集群中其他机器上的用户程序的副本。（那些运行用户程序的机器被称作worker，用于完成用户程序规定的的计算任务，worker本质上就是一个进程）

2. 所有worker中有一个特殊的worker是**master**，master主要负责任务的分配。通常会有M个map tast和R个reduce task，master会将这些任务分配给空闲的worker处理。理论上就会有M个map worker和R个reduce worker，但是实际上一个worker可以处理多个task，所以worker的数量一般都要少于task的数量。

3. 被分配**map**任务的worker开始**map阶段**的处理，读入一个split的输入数据，然后进行解析处理后输出一系列的中间键值对数据。中间键值对数据缓存在worker的内存中。

4. 每隔一段时间，缓存在内存中的中间键值对数据就会被写入到本地磁盘上，并且会对这些中间键值对执行**partition**操作划分成R份（partition函数：hash(key) mod R）。在map任务完成后，map worker会通知master这些中间键值对的存储位置，而master会将中间数据的位置信息转发给reduce worker。

5. map任务完成后，reduce worker从master处获取中间键值对的位置信息，然后进行remote read，随即开始**reduce阶段**。当reduce worker获取一个partition的中间键值对后，会将同一个key对应的键值对集中到一起（按key排序），再对每一个key都执行一次reduce函数。

6. 每次调用reduce函数的输出，都会被append到最终的输出文件中，每一个reduce worker都会产生一个输出文件。每个输出文件中包含了一部分计算结果，并且这些输出文件并不会被合并到一台机器上。（输出文件可以直接作为另一个计算任务输入数据，从而实现流水线的MapReduce计算，例如对网页进行词频统计后再对词频排序）

7. 当所有map任务和reduce任务都被完成后，程序通过MapReduce的return返回到用户代码。

#### master的数据结构

master会为每个map和reduce任务维护一个数据结构：

- task的状态：idle(尚未分配worker处理该task), in-progress, completed

- worker的id（非idle的task才会有）

master一个很重要的任务就是向reduce worker转发map产生的中间键值对的存储位置，而每个map worker会在完成它负责的map task后就向master汇报位置信息，master则会逐步向正在工作的reduce worker推送这些信息。

master不会等所有的map worker完成map task后再统一转发位置信息，而是有map任务完成后就会转发一次数据，以便尽早开始reduce阶段的处理，提高并行计算的效率

#### 错误容忍 - Fault Tolerance

master会周期性地ping所有的worker，如果没有在规定时间内获得应答，就会将worker标记为故障。

1. 对于in-progress的task，不管是map还是reduce任务，都会被重新标记为idle。task被标记为idle意味着需要重新安排worker来执行这个task。

2. 对于completed的task，只有map task需要别标记为idle被重新执行，因为map task的输出保存在本地磁盘，而reduce task的输出保存在GFS。

当有map task被重新执行的时候，所有reduce worker都会被通知，那些还没有读取该map task的输出结果的reduce worker都会读取map task重新执行后的输出结果。

#### 在故障存在的情况下的语义

**确定性的map和reduce操作（deterministic）**

        确定性的操作，意思是对于相同的输入会产生相同的输出。

        当用户提供的map和reduce函数是deterministic确定性时，MapReduce所执行的分布式计算的结果，与在一台机器上无错误顺序执行的结果一样。

        这需要确保map和reduce task的输出提交(commit)是原子性的：map会输出R个文件，reduce会输出1个文件，这些文件在task被完全complete之前，都只是执行task的worker所私有的，只有在task完成后，这些输出文件才会被命名为全局的名字，被master和其他worker所见。也就是说，一个task的输出要么是完整可见的，要么是不可见的，也就不存在不完整不正确的数据被其他worker或者用户获取的情况。

**非确定性的操作**

当用户提供的map和reduce函数是非确定性时，MapReduce也能提供相对合理的语义。

#### 位置优化

为了节省网络带宽，当master在分配map任务时，通常会选择那些在本地就存放有对应split的replica的机器，这样可以尽可能减少数据传输。

#### Task粒度

map阶段会被成M个map task，而reduce阶段有R个reduce task。理想情况下，M和R应该远大于worker的数量，也就是说，一个worker需要能够同时处理多个task。这会带来两个好处：

1. 提升动态的负载均衡：当一个worker的task处理得快的话可以接手其他worker的task

2. 当worker故障时，能够加速恢复：如果一个worker只处理一个task，那么当有worker故障时，该worker的task就只有等别的worker完成它负责的task后再来重新执行。

通常，M值的确定是需要确保划分输入文件后使得每个split大小在16~64MB，为了利用GFS特性提高效率。R值通常是worker数目的小的倍数。

#### Backup Task & Straggler

straggler可以理解为拖后腿的人 --- 当大多数worker的task任务都已完成时，还迟迟没有完成task的那些worker

当出现straggler时，会大大降低整个MapReduce任务的完成速度。因此，当MapReduce任务接近完成时，master会为那些还在执行的task安排备份任务，也就是再安排另外的task来同时执行这些任务，当primary task和backup task任何一个完成时，这个task就算完成。

### 1. What is shuffle?

<img src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-03-01-21-51-20-image.png" title="" alt="" width="507">

所有map产生的(k, v)列表，经过partition分组后，相同的k所对应的键值对会被集中到reduce worker上，交给reduce处理。这一把map输出的中间键值对分组再集中分配给reduce的过程就是shuffle。 

也就是上图中的过程：row（map的输出） -> column（reduce的输入）

### 2. GFS servers VS MapReduce servers

#### In 2004（time of the paper）

集群的网络结构如下，机器都连接到同一个以太网中，通过交换机组成局域网，局域网中有一个root交换机，机器之间的数据交换经常会需要经过这个root交换机，而当时的交换机所能支持的网络速率优先，所以需要尽量减少节点之间的通信。

<img src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-03-01-21-41-47-image.png" title="" alt="" width="384">

所以，in the paper，GFS server和MapReduce server会运行在同一台机器上，这样通过合理分配任务就可以实现paper中说的**locality optimization**，即worker直接从本地的GFS中读取输入文件，或者直接把输出文件保存在本地的GFS中。

#### Right Now

交换机的速率大大提升了，并且root交换机也不止一个，而是一组，这样就解决了root交换机处的网络速率瓶颈问题，从而对于整个局域网的网络吞吐量有了巨大提升。

<img src="file:///C:/Users/honor/AppData/Roaming/marktext/images/2024-03-01-21-57-22-image.png" title="" alt="" width="454">

所以，就没有必要通过**locality optimization**来提高效率，也没有必要把GFS server和MapReduce server放在一台机器上。两个服务分别运行在集群的不同节点上，这是modern MapReduce的普遍做法。
