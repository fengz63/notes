### 一、procfs概念
在许多的Unix计算机系统中，**procfs**是 进程文件系统（file system）的缩写，包含一个伪文件系统（启动时动态生成的文件系统），用于通过内核访问进程信息。这个文件系统通常被挂载到 ```/proc``` 目录。由于 ```/proc``` 不是一个真正的文件系统，它也就不占用存储空间，只是占用有限的内存。

### 二、Linux下/proc

Linux中的```/proc```实现了也克隆了九号项目中对应的部分。每个正在运行的进程对应于```/proc```下的一个目录，目录名就是进程的PID，每个目录包含：
+ /proc/PID/cmdline, 启动该进程的命令行.
+ /proc/PID/cwd, 当前工作目录的符号链接.
+ /proc/PID/environ 影响进程的环境变量的名字和值.
+ /proc/PID/exe, 最初的可执行文件的符号链接, 如果它还存在的话。
+ /proc/PID/fd, 一个目录，包含每个打开的文件描述符的符号链接.
+ /proc/PID/fdinfo, 一个目录，包含每个打开的文件描述符的位置和标记
+ /proc/PID/maps, 一个文本文件包含内存映射文件与块的信息。
+ /proc/PID/mem, 一个二进制图像(image)表示进程的虚拟内存, 只能通过ptrace化进程访问.
+ /proc/PID/root, 该进程所能看到的根路径的符号链接。如果没有chroot监狱，那么进程的根路径是/.
+ /proc/PID/status包含了进程的基本信息，包括运行状态、内存使用。
+ /proc/PID/task, 一个目录包含了硬链接到该进程启动的任何任务

Linux 2.6把```/proc```下大量的非进程相关的系统信息移动到一个专门的伪文件系统，称为```sysfs```（该文件系统是挂载到```/sys```上面）：
+ 电源管理系统（如果有的话）对应的目录/proc/acpi或/proc/apm
+ /proc/buddyinfo, 信息关于伙伴内存分配器用于处理内存碎片。
+ /proc/bus, 包含对应于计算机上各种总线的目录, 如input/PCI/USB. 在/sys/bus下包含更丰富的信息。
+ /proc/fb, 可利用的帧缓冲的列表
+ /proc/cmdline, 传递给内核的启动选项。
+ /proc/cpuinfo, 包含CPU信息, 诸如厂商（vendor），型号 (family, model，model names), 速度, 缓存大小, 逻辑核数 , 物理核数, CPU flags，以及BogoMips.