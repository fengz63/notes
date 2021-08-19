### 一. CPU Cache的作用
随着CPU速度的加快，CPU和内存再速度上的差异日趋显著。随着这些差异的扩大，继续引入一种新型的快速内存来弥补两者的差距。而这种新型的高速内存便是“CPU高速缓存（CPU Cache）”。

CPU cache是CPU用来降低从内存访问数据的平均成本（时间或性能）的硬件高速缓存。它位于存储层次结构体系（类似金字塔模型）中自顶向下的第二层，仅次于CPU寄存器。其容量远小于内存，但是读写访问速度接近CPU的时钟频率。如下图所示：

![](https://raw.githubusercontent.com/fengz63/picture/main/202108171435.jpg)

根据程序对内存地址访问规律上的局部性（时间局部性和空间局部性）原则，CPU会将最近频繁访问使用或是临近即将使用的指令或数据存储在cache中，以减少CPU下一次获取该指令的时钟周期。

当CPU在访问内存时候，会先去cache中查询所需指令或数据是否已存在，若存在，则不用去访问内存，取而代之的是直接获取并解码执行；若不在高速cache中，则继续向低一层级的内存中去获取该指令，载入高速cache，然后返回给cpu执行。

在操作系统中查询缓存大小和位置的命令如下所示：
```shell
$lscpu | grep cache
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              15360K
```
根据上述信息可知，该CPU体系结构中共有3级缓存，对于L1缓存，大小为64K，其中L1d是L1 Cache for Data，即数据缓存，L1i是L1 Cache for Instruction，即指令缓存。L2缓存大小为256K，L3缓存大小为15360K。

#### 1.1 Cache相关术语
1. 缓存命中（Cache Hit）：

    CPU需要数据时，先去L1搜索，若L1未找到，则接着L2、L3缓存中搜索，若找到了所需要的数据，则称为缓存命中。

2. 缓存缺失（Cache Miss）：

    若CPU在高速缓存中没有找到所的数据，则CPU必须请求将其从内存或存储设备（操作系统+虚拟存储器+虚拟内存范畴）加载到缓存，这便是缓存未命中。

3. 命中时间（His Rate）：

    访问某层存储器层次结构所需要的时间，包括了判断当前的访问是命中还是缺失所需要的时间。

4. 缺失代价（Miss Penalty）：

    将相应的数据（内存和cache间的块称为：高速缓存线）从低级存储器复制到高层存储器所需的时间。包括访问块、数据逐层传输、将数据插入发生缺失的层和将信息块传输给请求者的时间。

#### 1.2 Cache和Main Memory的异同
**相同点：**

+ 都是基于半导体和晶体管制造的；
+ Cache和Main Memory都是属于易失性存储器，当电源关闭时丢失其内容；

**不同点：**

+ Cache只保存主存种最常用的信息或程序代码的副本；
+ Cache通常集成在CPU芯片上。主内存（DRAM）放在主板上，并通过内存总线连接到CPU。
+ Cache更靠近CPU，因此读写速度比主存快
+ 主存比Cache大很多倍，而且造价更便宜。通常主存为几个G，而Cache则为几KB或几MB;

### 2. 什么是cache line
Cache Line能够简单的理解为CPU Cache中的最小缓存单位。内存和高速缓存之间或高速缓存之间的数据移动不是以单个字节或word完成的。

相反，移动的最小数据单位称为 **缓存行（Cache Line）**，有时称为缓存块。目前主流的CPU Cache的Cache Line大小都是64Bytes。假设有一个512字节的一级缓存，那么按照64B的缓存单位大小来算，这个一级缓存所能存放的缓存个数就是512/64 = 8个。

查询cache line指令如下所示：
```shell
$cat /sys/devices/system/cpu/cpu1/cache/index0/coherency_line_size
64
```

### 3. Cache映射
主存与cache的地址映射方式有直接映射、全相联方式和组相联方式三种。

具体见：[CPU Cache机制](https://www.shangmayuan.com/a/716ee5808b0b45428fbacbf0.html)

### 4. Cache Miss
当运算器须要从存储器中提取数据时，它首先在最高级的cache中寻找而后在次高级的cache中寻找。若是在cache中找到，则称为命中hit；反之，则称为不命中miss。

**Cache Miss的种类：**

1. Cold Miss：即在程序刚刚启动的时候，数据都是不在缓存中的，所以第一次访问数据必定发生 Miss ，这是不可避免的。
2. Conflict Miss：该Miss是由于不同的内存块映射到同一个cache set导致的。
3. Capacity Miss：容量性未命中，该Miss是由于程序运行所欲的set数量要大于缓存set的数量，导致不能把所有数据都装入缓存中。

更多参考: [Cache Miss与替换策略](https://blog.csdn.net/weixin_43895356/article/details/116606807)

### 5. 替换策略
出现 Miss 后，就需要在下一级的内存中读取数据，那么就有可能涉及 Cache Line 的替换，就涉及到了 CPU Cache 中的替换策略。

首先由于直接映射中每个地址映射到缓存中的位置都是唯一的，所以从下一级的内存中读取内存块到缓存中的策略就是直接替换

而组相联和全相联中一个内存块被映射到哪个 Cache Line 是不确定的，所以才存在替换策略将缓存中的一些 Cache Line 替换出去

+ 最不常使用（Least-Frequently-Used，LFU）：

    该策略会替换在过去某个时间窗口内引用次数最少的那一行。容易把新加入的块替换掉，且容易让前期频繁访问，后期较少访问的块长期驻留。

+ 最近最少使用（Least-Recently-Used, LRU)：
    
    策略会替换最后一次访问时间最久远的那一行。有较高命中率。

+ 随机替换
    
    随机选择一个 Cache Line 替换出去。随机替换算法在硬件上容易实现，且速度也比前两种算法快。缺点则是降低了命中率和Cache工作效率。

### 二、Page Cache
CPU如果要访问外部磁盘上的文件，需要首先将这些文件的内容拷贝到内存中，由于硬件的限制，从磁盘到内存的数据传输速度是很慢的。如果现在物理内存有剩余，会利用这些空闲内存来缓存一些磁盘的文件内容，这部分用作**缓存磁盘文件**的内存就叫做page cache。

此时，用户进程启动read()系统调用后，内核会首先查看page cache里有没有用户要读取的文件内容，如果有（cache hit），那就直接读取，没有的话（cache miss）再启动I/O操作从磁盘上读取，然后放到page cache中，下次再访问这部分内容的时候，就又可以cache hit，不用忍受磁盘的龟速了（相比内存慢几个数量级）。

#### 2.1 Page Cache 与 Buffer Cache
用一句话来解释，Page Cache用于缓存文件的页数据，Buffer Cache用于缓存块设备的块数据。页是逻辑上的概念，因此 Page Cache 是与文件系统同级的；块是物理上的概念，因此 buffer cache 是与块设备驱动程序同级的。

Page Cache 与 Buffer Cache的共同目的都是加速数据I/O：写数据时先写到缓存，将写入的页表纪委dirty，然后向外部存储flush，也就是缓存写机制中的write-back；读数据时首先读取缓存，如果未命中，再去外部存储读取，并且将读取来的数据也加入缓存。操作系统总是积极地将所有空闲内存都用作 Page Cache 和 Buffer Cache，当内存不够用时也会用 LRU 等算法淘汰缓存页。

更多见：[Linux的Page Cache](https://spongecaptain.cool/SimpleClearFileIO/1.%20page%20cache.html)

#### 2.2 Page Cache的优缺点
1. Page Cache的优势：
    + 加快数据访问：
        
        如果数据能够在内存中进行缓存，那么下一次访问就不需要通过磁盘I/O了，直接命中内存缓存即可。由于内存访问比磁盘访问快很多，因此加快数据访问时Page Cache的一大优势
    + 减少I/O次数，提高系统磁盘I/O吞吐量

        得益于Page Cache的缓存以及预读能力，而程序又往往符合局部性原理，因此通过一次I/O将多个page装入Page Cache能够减少磁盘I/O次数，进而提高系统磁盘 I/O 吞吐量。

2. Page Cache的缺点：
    + 最直接的缺点是需要占用额外物理内存空间，物理内存在比较紧俏的时候可能会导致频繁的 swap 操作，最终导致系统的磁盘 I/O 负载的上升。
    + 另一个缺点是对于应用层并没有提供很好的管理 API，几乎是透明管理
    + 最后一个缺陷是在某些应用场景下比 Direct I/O 多一次磁盘读 I/O 以及磁盘写 I/O。