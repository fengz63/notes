### 一、netstat命令
netstat命令用于显示各种网络相关的信息，如网络连接，路由表，接口状态（Interface Statistics），多播成员（Multicast Memberships）等。

执行netstat命令后，其输出结果如下所示：
```shell
$netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 e18c13440.et15sqa:21157 100.67.147.36:http      TIME_WAIT
tcp        0      0 e18c13440.et15sqa:49987 10.101.0.55:8346        ESTABLISHED
tcp        0    236 e18c13440.et15sqa:ssh   30.225.32.126:54138     ESTABLISHED
tcp        0      0 e18c13440.et15sqa:32376 100.67.160.129:http     TIME_WAIT
tcp        0      0 e18c13440.et15sqa:58696 100.67.148.220:http     TIME_WAIT
tcp        0      0 e18c13440.et15sqa:30755 alimonitor1.et15s:19888 ESTABLISHED
tcp        0      0 e18c13440.et15sqa:33584 100.67.213.129:http     ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  6      [ ]         DGRAM                    41       /run/systemd/journal/socket
unix  14     [ ]         DGRAM                    43       /dev/log
unix  2      [ ]         DGRAM                    49521    /var/run/chrony/chronyd.sock
unix  2      [ ]         DGRAM                    975      /run/systemd/shutdownd
```

从整体上看，netstat的输出结果可以分为两个部分：

+ 一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。

+ 另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。

Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。

**状态说明：**

通过netstat命令可以查看到网络的状态一共有11种，如下所示：

```
LISTEN：侦听来自远方的TCP端口的连接请求
SYN-SENT：再发送连接请求后等待匹配的连接请求（如果有大量这样的状态包，检查是否中招了）
SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（如有大量此状态，估计被flood攻击了）
ESTABLISHED：代表一个打开的连接
FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认
FIN-WAIT-2：从远程TCP等待连接中断请求
CLOSE-WAIT：等待从本地用户发来的连接中断请求
CLOSING：等待远程TCP对连接中断的确认
LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认（不是什么好东西，此项出现，检查是否被攻击）
TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认
CLOSED：没有任何连接状态
```

更多参考：[Linux netstat命令详解](https://cloud.tencent.com/developer/article/1593886)

### 二、ping命令
在诊断网络问题时，我们经常会使用ping命令。它可以快速告诉我们，某个域名是否可以可以访问，访问延时高不高。

#### 2.1 实现原理
ping命令主要基于ICMP（Internet Control Message Protocol）实现，它包含了两部分：客户端和服务器：
+ 客户端：向服务端发送ICMP回显请求报文（echo message）
+ 服务端：向客户端返回ICMP回显响应报文（echo reply message）

#### 2.2 ping命令过程

机器 A ping 机器B

**同一网段：**

+ ping 通知系统建立一个固定格式的 ICMP 请求数据包
+ ICMP 协议打包这个数据包和机器 B 的IP地址转交给 IP 协议层
+ IP 层协议将以机器B的IP地址为目的地址，本机IP地址为源地址，加上一些其他的控制信息，构建一个IP数据包
+ 获取B的mac地址

IP层协议通过机器B的IP地址和自己的子网掩码，发现它跟自己属同一网络，就直接在本网络查找这台机器的MAC。

若两台机器之前有过通信，在机器A的ARP缓存表应该有B机IP与其MAC的映射关系；若没有，则发送ARP请求广播，得到机器B的MAC地址，一并交给数据链路层

数据链路层构建一个数据帧，目的地址是IP层传过来的MAC地址，源地址是本机的MAC地址，再附加一些控制信息，依据以太网的介质访问规则，将他们传送出去。

机器B收到这个数据帧后，先检查目的地址，和本机MAC地址对比，如果符合，接收。接收后检查该数据帧，将IP数据包从帧中提取出来，交给本机的IP协议层协议。IP层检查后，将有用的信息提取交给ICMP协议，后者处理后，马上构建一个ICMP应答包，发送给主机A，其过程和主机A发送ICMP请求包到主机B类似（这时候主机B已经知道了主机A的MAC地址，不需再发ARP`请求）；不符合，丢弃。

**不同网段：**
+ ping 通知系统建立一个固定格式的 ICMP 请求数据包
+ ICMP协议打包这个数据包和机器B的IP地址转交给IP协议层
+ IP层协议将以机器B的IP地址为目的地址，本机IP地址为源地址，加上一些其他的控制信息，构建一个IP数据包
+ 获取主机B的mac地址

IP协议通过计算发现主机B与自己不在同一网段内，就直接交给路由处理，就是将路由的MAC取过来，至于怎么得到路由的MAC地址，和之前一样，先在ARP缓存表中寻找，找不到可以利用广播。路由得到这个数据帧之后，再跟主机B联系，若找不到，就向主机A返回一个超时信息。

### 三、ARP协议
#### 3.1 什么是ARP协议
网络层以上的协议用IP地址来标识网络接口，但以太数据帧传输时，以物理地址来标识网络接口。因此我们需要进行IP地址与物理地址之间的转化。对于IPv4来说，我们使用ARP地址解析协议来完成IP地址与物理地址的转化（IPv6使用邻居发现协议进行IP地址与物理地址的转化，它包含在ICMPv6中）.ARP协议提供了网络层地址（IP地址）到物理地址（mac地址）之间的动态映射。ARP协议 是地址解析的通用协议。

#### 3.2 ARP协议工作流程
+ 每个主机都会在自己的 ARP 缓冲区中建立一个 ARP 列表，以表示 IP 地址和 MAC 地址（以太网地址）之间的对应关系。

+ 主机（网络接口）新加入网络时（也可能只是MAC 地址发生变化，接口重启等）， 会发送免费ARP报文把自己IP地址与Mac地址的映射关系广播给其他主机。

+ 网络上的主机接收到免费ARP报文时，会更新自己的ARP缓冲区。将新的映射关系更新到自己的ARP表中。

+ 某个主机需要发送报文时，首先检查 ARP 列表中是否有对应 IP 地址的目的主机的 MAC 地址，如果有，则直接发送数据，如果没有，就向本网段的所有主机发送 ARP 数据包，该数据包包括的内容有：源主机 IP 地址，源主机 MAC 地址，目的主机的 IP 地址等。

+ 当本网络的所有主机收到该 ARP 数据包时：

    首先检查数据包中的IP 地址是否是自己的 IP 地址，如果不是，则忽略该数据包。

    如果是，则首先从数据包中取出源主机的 IP和 MAC 地址写入到 ARP 列表中，如果已经存在，则覆盖。
    
    然后将自己的 MAC地址写入ARP响应包中告诉源主机自己是它想要找的MAC地址。

+ 源主机收到 ARP 响应包后。将目的主机的 IP 和 MAC 地址写入 ARP 列表，并利用此信息发送数据。如果源主机一直没有收到 ARP 响应数据包，表示 ARP查询失败。

更多参考：[20张图解：ping的工作原理](https://www.cnblogs.com/xiaolincoding/p/12571184.html)