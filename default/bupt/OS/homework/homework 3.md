# 结果含义
## Basic system parameters
- **Host**: 测试机器的标识符。
- **OS Description**: 操作系统信息，比如内核版本、架构。
- **Mhz**: CPU 主频。
- **tlb pages**: TLB 可缓存的虚拟页数，影响虚拟地址转换性能。
- **cache line bytes**: CPU cache 一行的大小（通常是 64/128 字节）。
- **mem par**: 内存访问的并行度指标，越大说明能同时访问更多内存块。
- **scal load**: 表明系统在不损失性能的前提下的负载能力。
## Processor, Processes
- **null call**: 调用空函数的开销。
- **null I/O**: 空的 I/O 调用开销。
- **open/close**: 打开/关闭文件的时间。
- **slct TCP**: `select()` 系统调用在 TCP socket 上的延迟。
- **sig inst**: 注册一个信号处理函数的时间。
- **sig hndl**: 信号触发并进入处理函数的延迟。
- **fork proc**: 使用 `fork()` 创建子进程的时间。
- **exec proc**: 使用 `exec()` 启动新程序的时间。
- **sh proc**: 启动一个 shell 的时间。
## Basic integer operations
- **intgr bit**: 基本的位操作。
- **intgr add**: 整数加法。
- **intgr mul**: 整数乘法。
- **intgr div**: 整数除法。
- **intgr mod**: 整数取模。
## 64 位、float、double 操作
和上面一样，只不过是**不同数据类型**的运算。
## Context switching
测试不同进程数和不同工作集大小时，**上下文切换**的时间。这反映了 CPU 在进程切换时保存和恢复寄存器/缓存的开销。
## Communication latency
- **pipe**: 管道通信的延迟。
- **AF UNIX**: Unix socket 延迟。
- **TCP**: TCP 延迟。
- **RPC/UDP/TCP**: 远程过程调用在不同协议下的延迟。
## File & VM system latencies 
- **0k File Create/Delete**: 创建/删除小文件的时间。
- **10k File Create/Delete**: 创建/删除大文件的时间。
- **Mmap Latency**: 映射内存的延迟。
- **Prot Fault**: 内存保护错误触发的延迟。
- **Page Fault**: 缺页中断的处理时间。
## Communication bandwidth
- **pipe**: 管道通信的吞吐量。
- **AF UNIX**: Unix 域 socket 的吞吐量。
- **TCP**: 本地 TCP socket 的吞吐量。
- **File reread**: 文件重新读取时的吞吐量（受缓存影响）。
- **Mmap reread**: 使用内存映射文件的吞吐量。
- **bcopy**: 内存块拷贝速度。
- **Mem read/write**: 内存的读/写速度。
## Memory Latency
- **L1 cache latency**: 一级缓存的访问延迟。
- **L2 cache latency**: 二级缓存延迟。
- **Main memory latency**: 内存访问延迟。
- **Random access latency**: 随机访问的延迟。
---
# My Test
选择了 open/close，fork，exec 三个系统调用进行测试，结果如下：
```
Better testing open/close speed...
Average time for better open/close: 850.352762 microseconds
Better testing fork speed...
Average time for better fork: 148.900975 microseconds
Better testing exec speed...
Average time for better exec: 385.518957 microseconds
```
LMBench 的结果如下（微秒）：

| open/close | fork | exec |
| ---------- | ---- | ---- |
| 1.16       | 141  | 319  |
| 1.10       | 136  | 328  |
可以发现，fork 和 exec 略高于 LMBench 而 open/close 过于超出，分析可能原因如下：
1. LMBench 测试的 open/close 应该只测试了这两个操作单独所消耗的时长，而在我的测试中，调用 `open` 和 `close` 函数也许包含其他复杂操作例如寻找文件等等，导致时间大幅度增加
2. 我使用了 for 循环进行多次测试，其中还包含条件判断语句，这些都有可能增加操作时间，导致 fork 和 exec 也有时间增加
3. exec 所执行的程序也可能与 LMBench 所选择的有所不同，我还包括了程序执行的时间，不单纯是 exec 所花费的时间，exec 中也包含 fork 操作，所以总体上 exec 的时间增加量大于 fork
## 怎么达到 LM 一样的效果？
需要尽可能地删去不必要的开销。
主要做出了两个优化：
### tmpfs
将文件创建在 tmpfs/RAM-backed filesystem 中，使得文件存储在 RAM 里，不需要进行 disk IO 操作，这样所有的操作都发生在 memory 里面。
### 预热
在正式开始测试之前，先多次重复打开文件，这样文件就被存放在了缓存中，同样可以避免开销巨大的磁盘操作。
### 最终结果
```
Best testing open/close speed...
Average open+close: 0.862 microseconds
```
成功达到 LM 的数量级。