### 一、文件描述符
文件描述符（File Descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。

文件描述符在形式上是一个非负整数。实际上，**它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表**。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。

一般来说，每个进程最多可以打开64个文件。在不同的系统上，最多允许打开的文件个数不同，Linux 2.4.22 强制规定最多不能超过 1,048,576。

每个进程默认都有 3 个文件描述符：0 (stdin)、1 (stdout)、2 (stderr)。

**Socket与fd的关系：**

socket 是 Unix 中的术语。socket 可以用于同一台主机的不同进程间的通信，也可以用于不同主机间的通信。一个 socket 包含地址、类型和通信协议等信息，通过 socket() 函数创建：
```cpp
int socket(int domain, int type, int protocol)
```
返回的就是这个socket对应的文件描述符fd。操作系统将socket映射到进程的一个文件描述符上，进程就可以通过读写这个文件描述符来和远程主机通信。

可以这样理解，socket是进程间通信规则的高层抽象，而fd提供的是底层的具体实现。socket和fd是一一对应的。通过socket通信，实际上就是文件描述符fd读写文件。

**fd_set文件描述符集合：**

参数中的fd_set类型表示文件描述符的集合。

由于文件描述符fd是一个从0开始的无符号整数，所以可以使用```fd_set```的**二进制每一位**来表示一个文件描述符。某一位为1，表示对应的文件描述符已就绪。比如设fd_set长度为1字节，则一个fd_set变量最大可以表示8个文件描述符。当select返回```fd_set=00010011```时，表示文件描述符1，2，5已经就绪。

fd_set的使用涉及以下几个API：
```cpp
#include <sys/select.h>   
int FD_ZERO(int fd, fd_set *fdset);  // 将 fd_set 所有位置 0
int FD_CLR(int fd, fd_set *fdset);   // 将 fd_set 某一位置 0
int FD_SET(int fd, fd_set *fd_set);  // 将 fd_set 某一位置 1
int FD_ISSET(int fd, fd_set *fdset); // 检测 fd_set 某一位是否为 1
```

### 二、IO模式
缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

**缓存IO的缺点：**

数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

#### 2.1 IO多路复用
select，poll，epoll都是IO多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

1. select

    select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符。函数签名如下所示：
    ```cpp
    int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
    ```

    select的缺点：
    + 性能开销大：
        + 调用select时会陷入内核，这时需要将参数中的fs_set从用户空间拷贝到内核空间
        + 内核需要遍历传递进来的所有fs_set的每一位，不管他们是否就绪。
    + 同时能够监听的文件描述符数量太少。受限于sizeof(fd_set)的大小，在编译内核时就确定了且无法更改。一般是1024，不同的操作系统不相同。

2. poll

    poll和select几乎没有区别。poll采用链表（这里存疑，Linux文档中貌似还是数组）的方式存储文件描述符，没有最大数量的限制。

    poll的函数签名如下所示：
    ```cpp
    int poll(struct pollfd *fds, nfds_t nfds, int timeout);
    ```
    其中fds是一个pollfd结构体类型的数组，调用poll()时必须通过nfds指出数组fds的大小，即文件描述符的数量。详见：[manage-poll(2)](https://man7.org/linux/man-pages/man2/poll.2.html)

    从性能开销上看，poll和select的差别不大。

3. epoll

    epoll是对select和poll的改进，避免了“性能开销大”和“文件描述符数量少”两个缺点。简而言之，epoll有以下几个特点：
    + 使用**红黑树**存储文件描述符集合
    + 使用**队列**存储就绪的文件描述符
    + 每个文件描述符只需在添加时传入一次；通过事件更改文件按描述符状态

    select、poll 模型都只使用一个函数，而 epoll 模型使用三个函数：```epoll_create```、```epoll_ctl``` 和 ```epoll_wait```。

    epoll的优点：
    + 对于“文件描述符数量少”，select 使用整型数组存储文件描述符集合，而 epoll 使用红黑树存储，数量较大。
    + 对于“性能开销大”，epoll_ctl 中为每个文件描述符指定了回调函数，并在就绪时将其加入到就绪列表，因此 epoll 不需要像 select 那样遍历检测每个文件描述符，只需要判断就绪列表是否为空即可。这样，在没有描述符就绪时，epoll 能更早地让出系统资源。（相当于时间复杂度从O(n)降为O(1)）
    + 此外，每次调用 select 时都需要向内核拷贝所有要监听的描述符集合，而 epoll 对于每个描述符，只需要在 epoll_ctl 传递一次，之后 epoll_wait 不需要再次传递。这也大大提高了效率。

更多具体见：[I/O 多路复用，select / poll / epoll 详解](https://imageslr.com/2020/02/27/select-poll-epoll.html)