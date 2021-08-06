### linux启动过程
关于linux系统的启动流程可以分为以下步骤：
POST（加电自检）->加载BIOS（Basic  Input/Outpu System)–>确定启动设备（Boot sequence)、加载Boot Loader–>加载内核（kernel）初始化initrd–>运行/sbin/init初始化系统–>打印用户登录提示符

#### 1. POST开机自检
linux开机加电后，系统开始开机自检，该过程主要对计算机的各种硬件设备进行检测，如CPU、内存、主板、硬盘、CMOS芯片等。
+ 如果出现致命故障则停机，并且由于初始化过程还没完成，所以不会出现任何提示信号；
+ 如果出现一般的故障则会发出声音等提示信号，等待故障清除；若未出现故障，加电自检完成。

#### 2. 开机自检完成，查找可启动设备，加载主引导目录（MBR）
开机自检完成后，CPU首先读取位于CMOS中的BIOS程序，按照BIOS中设定的启动次序（Boot Sequence）逐一查找可启动设备，找到可启动设备后，去该设备的第一个扇区中读取MBR。什么是MBR呢？

MBR存在于可启动磁盘的0磁道0扇区，占用512 字节，它主要用来告诉计算机从选定的可启动设备的哪个分区来加载 引导加载程序（Boot loader），MBR中存在如下内容：

（1）Boot Loader占用 446 字节，存储有操作系统相关信息，如操作系统名称，操作系统内核位置等，它的主要功能是加载内核到内存中运行。

（2）Partition Table 分区表，占用 64 字节，每个主分区占用16字节（这就是为啥一块硬盘只能有4个主分区的原因）。

（3）分区表有效性标记占用 2 字节。

CPU将MBR读取至内存，运行GURB（Boot Loader常用的有 GRUB、GURB2 和 LILO 三种，现在常用的是GRUB2），GRUB会把内核加载到内存去执行。

![](https://raw.githubusercontent.com/fengz63/picture/main/20210805192300.jpg)

上图中的```vmlinuz-5.10.23-5.al8.x86_64```即为内核文件。从上图中可以看出，内核文件存在于 /boot 目录下，但是在 GRUB 加载内核时，连 / 都还没有被加载，它是怎么在磁盘上找到内核呢？首先查看下GRUB的配置文件，如下所示：

具体见：[linux启动过程](https://cloud.tencent.com/developer/article/1114481)

总结一下，grub启动过程可以分为两个步骤：
+ 第1阶段：BIOS加载MBR中的GRUB（GRUB）第一阶段文件，而GRUB只有446字节，无法实现太多功能，所以利用该阶段的文件去加载1.5阶段的文件（/boot/grub/下的文件）
+ 第1.5阶段：用来加载识别文件系统的文件，识别完系统后才可以找到/boot目录。
+ 第2阶段：寻找内核并加载到内存中

#### 3. 加载内核，初始化 initrd
GRUB把内核加载到内存后展开并运行，此时GRUB的任务已经完成，接下来内核将会接管并完成 探测硬件–>加载驱动–>挂载根文件系统–>切换至根文件系统（rootfs）–>运行/sbin/init完成系统初始化。

init 始终是第一个要执行的程序，并被分配进程 ID 或 PID 为 1。它是 init 进程，它产生各种守护进程并挂载/etc/fstab文件中指定的所有分区。

内核然后挂载初始 RAM 磁盘 (initrd)，它是一个临时的根文件系统，直到真正的根文件系统被挂载。所有内核都与初始 RAM 磁盘映像一起位于 /boot 目录中。

#### 4. 运行 Systemd
内核最终加载Systemd，它取代了旧的SysV init。Systemd是所有Linux进程之母，它管理文件系统的挂载、启动和停止服务等等。

Systemd 使用 /etc/systemd/system/default.target，以确定linux系统应该引导到的状态或目标。

以下是systemd目标的细分：
+ poweroff.target(runlevel 0): 关闭或关闭系统
+ rescue.target(runlevel 1): 启动救援shell会话
+ multi-user.target(runlevel 2,3,4): 将系统配置为非图形（控制台）多用户系统
+ graphics.target(runlevel 5): 将系统设置为使用具有网络服务的图形多用户界面
+ reboot.target(runlevel 6): 重新启动系统

#### 5. 打印登录提示符
一旦systemd加载所有的守护进程并设置目标或运行级别值，启动过程就结束了。此时，系统会给出录提示符（login）或者图形化登录界面，用户输入用户和密码登陆后，系统会为用户分配一个用户ID（uid）和组ID（gid），这两个ID是用户的 身份标识，用于检测用户运行程序时的身份验证。登录成功后，整个系统启动流程运行完毕！