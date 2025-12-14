# Lecture 1 Intro
## What is OS?
A bridge between hardware and apps/users. Or, a special software layer that provides and manages the access from apps/users to hardware resources (CPU, memory, disk, etc).
# Lecture 2 Boot, Process, Kernel
## How computers boot?
### BIOS
Basic Input Output System, it is a firmware, it is responsible for identify attached hardware and initialize their states, build hardware description for advanced configuration and power interface, load a **boot-loader** from disk to memory then transfer the control to the boot-loader to do some stuff in the memory.
### Boot-loader
It is part of the OS, since it is running on memory. It will switch from **real mode** to **protected mode**. This is a historical reason, in a word, real mode means less powerful OS, and protected means modern OS, it is designed for compatibility. After that the boot-loader will check the if kernel image is OK, then load it from disk to memory. Finally, it transfer control to real "OS".
### Question
Why do we need 2 pass? Why can't BIOS load OS directly? This is due to flexibility and compatibility. BIOS is kind of "hard-coded" while boot-loader is easier to change. Separate process into pieces also makes errors easier to handle.
### UEFI
Unified Extensible Firmware Interface. A successor of BIOS, which is more modern.
## Process
**Definition**: the execution of an application program with restricted rights. It differs from program, it is an instance of a program, like an object is an *instance* of a class in Object Oriented Programming.
### Process Control Block
**Definition**: PCB is a data structure used by Linux to keep track of a process execution.
### Process In Memory
An executable mainly consist of bss(Block Starting by Symbol), data and code regions. And the running process will have its own stack and heap. Uninitialized data are stored in bss region and initialized data are stored in data region.
## Dual Mode
### How to Virtualize?
The basic model of CPU virtualization: run one process for a little while, then run another one, and so forth.
This leads to 2 challenges:
1. **Performance**: how to virtualize without adding excessive overhead to the system?
2. **Control**: OS must take control whenever it wants;otherwise a process can run forever. OS must control how certain resources can be accessed by processes. We can just let OS take charge of every instruction of process, but this seems like we are running them on a Virtual Machine to test whether it is safe then actually run it, which is too slow.

So rather than always direct run or always go through OS, we perform the philosophy of Computer Science, absorbing the advantages. We design a method that **"sensitive" instructions go through OS and most instructions directly run on CPU**. The "sensitive" instructions are predefined by human, what are they?
### Limited Direct Execution
OS needs to check the instructions to be executed, in order to prevent detrimental behavior.
The following instructions are limited:
1. Restricted operations;
2. inter-process switching: voluntary switching like: system calls and involuntary switching like: timer interrupt (running for too long).

The next problem is how to implement?
### Dual Mode
User mode (用户态) and kernel mode (内核态) 这两者都是 CPU 的状态，由 OS 控制。
The status is stored in `cs` register, 0 for kernel mode, 3 for user mode (use only 2 bits).
### Privileged Instructions (特权指令)
I/O read/write, context switch, changing privilege level, set system time... Any instructions that *could affect other processes* are likely to be privileged.
### Memory Protection
Segmentation: base and bound.
Paging: every memory address a process sees is *"discontinuously"* mapped to a physical address in memory.
The kernel code is usually placed at high address, but not low address, this is to prevent user accidentally wants to access kernel code and we can separate kernel code and user with the prefix of address easily.
# Lecture 3 Context Switch
## User to Kernel Mode Switch
### Exceptions
When processor encounter *unexpected* condition.
e.g. divide-by-zero, perform privileged instructions, etc.
### Interrupts
*Asynchronous* signal to the processor that some *external* event has occurred.
### System calls
User processes *request* the kernel to do some operations.
### Interrupt Vector Table (中断向量表)
It stores the entries of different handlers for exceptions, interrupts and traps (all of three, the name is confusing). There is a register pointing to the vector table which is stored in kernel memory, and each element in the vector points to handler.
Nowadays we have something like Interrupt Descriptor Table (IDT, 中断描述符表), which tells CPU where the Interrupt Service Routines (ISR, 中断服务程序) are located.
*Entries* are called "**Gates**" (the gate that separates user and kernel), its location is kept in **IDTR** (IDT register) and is loaded with **LIDT** assembly instruction.
### Interrupt Masking (中断屏蔽)
*Disable interrupts* and *enable interrupts* are two privileged instructions. This means some interrupts can be masked out during processing some important things, but there are still some most important things. So, we have:
- **Maskable** interrupts: all software interrupts, all system calls, and partial hardware exceptions
- **Non-Maskable** interrupts: partial hardware exceptions

The masked interrupts are actually deferred but not ignored.
### Interrupt Stack (中断栈)
**Definition**: a special stack in kernel memory to store interrupt status.
We have **one** for **each** process (thread). When first fault, we trap into kernel-exception handler. Second fault, we trap to another handler. Triple fault, we reboot.
We let each thread has one interrupt stack to make it easier to switch a new process inside an interrupt or system call handler.
## x86 Mode Transfer
When an interrupt/exception/system call occurs, the OS will:
1. Save the rest of the interrupted process's state with `pusha` or `pushad`
2. Execute the handler
3. Resume the interrupted process with `popa` or `popad` and pop the error code
4. Resume the interrupted process with `iret`
# Lecture 4 OS Interfaces and System Calls
## OS Programming Interface
We have talked about a lot of upcalls to OS from apps, and we have Portable Operating System Interface (POSIX), which is a standard for UNIX likes systems, especially about its system calls.
Above POSIX, we have libc, which is an overview of standard C libraries. It is mainly consist of POSIX API and standard C functions. If a software is only written in libc, then it has good portability across OS/hardware.
For example, the code we write that calls libc functions will call POSIX API to do further stuff.
Such design that thins the **System Call Interface** make sure that each system call is limited to its own functionality, so that app designers and system designers both can mix and use these primitives. This provides better flexibility, safety, reliability and performance.
## Process Management
In WINDOWS you can use `CreateProcess` to create a process that runs specified program, but who cares WINDOWS, let's see the process management in UNIX.
### fork
Create a complete copy of the parent process, of course, with new address space, but all the memory contents and execution contents of parent are inherited. Another difference is the returning value of `fork()`, it will return 0 in child process and the process id of child process in parent process.
### exec
Load the program specified into current address space, it does not create new process. So the typical usage of `exec()`is that we first `fork()` a new process then `exec()`.
## Input/Output
Computer systems has very diverse I/O devices, keyboards, disk, network, mouse, etc. If we design one interface for each devices, then whenever a new device is added we will have to design a new interface, which will be hard to expand and maintain.
So UNIX has **one interface for all of them**! UNIX think every thing as a file, we can open, read, write, close it.
### File Descriptor
**Definition**: `fd`, a number `int` type, that uniquely identifies an open file in a computer's operating system. It describes a data resource, and how that resource can be accessed.
With `fd` and a kernel buffer with two `fd`, we call this buffer pipe, we can extend the interface to inter-process communication. Suppose parent opened two `fd` in pipe, then `fork()` a child, the child will inherit the opened `fd`, they can perform communication with the read and write on pipe.
## System Call Design
An illusion is that kernel is simply a set of library routines like normal libraries. Actually, it's not, the code are not even in the same context! This is because of the key challenge in Operating Systems: we don't trust user, we need to **protect system from user-space errors.**
Since then, even though kernel can directly access user parameters without copying (they are all in memory), they still need to copy it to prevent modification in user memory stack and for safety. The parameters are also checked before doing the handling, but not before copying them to kernel memory. This is in order to prevent **time of check, time of use attack**, which means, the parameters may passed the check, but there is a small time gap between the actual use of the data and the check time, so the malicious attackers may still be able to modify the data. This is why we should check the copied data in kernel.
## Aside
There is difference between `Ctrl-C` and `Ctrl-Z`, though they both can stop a process. `Ctrl-C` sends a `SIGINT` (interrupt) to the process, while `Ctrl-Z` sends a `SIGTSTP` (stop) signal that pauses the process in mid-execution, which means you can continue it afterwards.
# Lecture 5 Threads
Introduce two concepts first.
**Concurrency**: *dealing* with a lot of work at once, one worker, multiple tasks.
**Parallel**: *doing* a lot of work at once, multiple worker, multiple tasks.
Concurrency is about what does one core do, parallel is about what does multiple cores do.
## Thread Abstraction
**Definition**: thread is a single *execution sequence* that represents a *separately schedulable task*. It is the **minimal scheduling unit in OS**.
Threads in the same process share memory space like: code, data, files, but has their own execution context like: registers and stack.
Thread execution speed in "unpredictable", you may switch to other thread at any given time, so how to handle such randomness is a really hard part in multi-thread programming.
## Thread Implementation
### Thread Data Structure
Thread Control Block (TCB):
- Stack pointer, each thread has their own stack
- Copy of processor registers
- Metadata

It does not have code, data, file information like PCB. These are the **shared states**:
- Code
- Global Variables
- Heap Variables

To delete a thread, we can remove it from the ready list, so that it will never run again, we also need to free the per-thread state allocated for the thread. But how to free the space? We can't let the thread itself to do so, its just like you need to kill yourself then you also need to say "Hey, I'm gonna quit!", without a stack, the thread can do nothing. So, we add the deleted threads to a *finished* threads list, their space will be freed by *other* threads.
# Lecture 6 Address Translation
## Concept
As the name suggests, address translation translate from virtual memory address to physical memory address. If the physical address is valid, it will access it and work, if not valid, throw an exception.
The **GOAL** and motivation of address translation: memory protection, memory sharing, flexible memory placement, sparse addresses, runtime lookup efficiency, compact translation tables, portability, etc.
## Segmentation
**Simplest Approach**: *base* and *bounds* registers, every memory access is checked on those registers. We will need a segment table to record these bases, bounds and read write permissions.
However, it reality, we find the usage of physical memory is fragmented, there are a lot of "holes", this is because process comes and goes, not every process can fit in spare space. Many methods are designed to enable fine-grained allocation.
In real mode (for anyone who forgets about real mode, it is just like kernel mode), there is no segment table, we shift left the segment register value and add offset value to it.
In protected mode, the segment table is called global descriptor table (GDT) or local descriptor table (LDT). Linear address = base address + offset.
Segmentation is easy to implement quite straight forward, but the principle downside is: overhead of managing a large number of variable size and dynamically growing memory segments, it would be easy if you are satisfied with a bad strategy. And that will lead to *external fragmentation*, free space becomes non-contiguous. Compacting the memory will be slow, and even more complex if the segments can grow.
## Paging
**Definition**: allocating memory in *fixed-sized* chunks called *page frames*. We have a *page table* stores for each process whose entries contain pointers to the page frames.
Page table is more compact that segment table because it does not need to store bounds. And now the pages are scattered across physical memory regions, yet with each page, the memory access is still contiguous. The memory allocation becomes very simple, we only need to find a page frame.
### How does it work?
Since we have fixed page size, suppose we have a 4 KB page size (the most common one), then we let the lowest 12 bits of our virtual address be the offset, and use the first 20 bits (suppose we have 32 bits address) to locate the frame id in the page table. Then we combine the frame id and offset to get the physical address. This is also the only way to get physical address: combine frame id in page table with offset. This will be mentioned again later.
### How large is the page table?
Each entry in the page table takes 32 bits, that is 4 bytes and we have $2^{20}$ entries, that will be 4 MB. Remember we have a page table for each process, so now open your task manager and you will find out that most processes only need around 0.1 MB! What a waste if I need 4 MB page table to support such a tiny process! And this is the main disadvantage of page table method.
### Multi-Level Paging
To solve such overhead, we introduce multi-level paging, **page directory**.
Separate the address into 3 parts: **page directory number** (10 bits), **page table number** (10 bits), **page offset** (12 bits). We use page directory number and page directory register `CR3` (tells you which directory to visit) to locate which entry in the page directory to visit, which is the address of page table, then we use page table number to locate the frame id. Finally combine frame id with offset to get physical address.
Note that page directory and page table only takes 4 KB now. So for processes that only needs small space, the cost of page directory and page table now reduces to 8 KB. The philosophy behind such improvement is to make page table finer-grained.
We can even extend such multi-level to 3 or more layers.
___
Memory management unit (MMU), a hardware, does such translation for us.
The page size shall be neither too small or too large. Too small will lead to large page table size and low cache hit ratio, too large will lead to internal fragmentation (frame is not fully used), typically page size range from 512 B to 8192 B. Default 4 KB on Linux. Page tables can be sparse, not every page directory entry has a corresponding page table, this saves a lot of space.
It is also worth mentioning that why it takes 10 bits to decide the entries, it is because we want to fit the directory and table into one frame. So it is actually the frame size decides how the process of translation with page table is done. Refer to the OSTEP book for more info.
### Page Fault
**Definition**: page fault happens when CPU/MMU access a memory location that is not readily mapped:
- Pure (soft): memory swapped out; shared pages; etc. After handled, the access will be performed again.
- Invalid (hard): write to read-only pages; across to pages not allocated; etc. This leads to Segmentation Fault!

In modern OS, `malloc` does memory allocation "lazily", which means:
- It allocates virtual memory immediately
- The physical memory is allocated only when program accesses the memory through page fault handler
- This enables constant time allocation and unlimited resource illusion
# Lecture 7 TLB and cache
## Cache Concept
**Definition**: cache is a repository for copies that can be accessed more quickly that the original
Average Access Time = Hit Rate$\times$Hit Time+Miss Rate$\times$Miss Time
### Locality
The key to cache success is locality, we have two kinds:
- **Temporal locality**: the memory is accessed is more likely to be accessed again, e.g. iterating variable.
- **Spatial locality**: when a particular storage location is referenced at a particular time, then its likely that near by memory locations will be referenced soon, e.g. scanning through an array.
### Memory Hierarchy
From locality we can infer that we can make system much faster by putting those memory value with good locality near the CPU. This leads to the introduction of **Memory Hierarchy**.
In this course we will learn about: CPU, TLB, Cache and Memory. The disk is the slowest.
## TLB
TLB derives from address translation. Each memory access will take at least 2 extra memory access to page directory and page table. And memory speed is often slower that CPU, when we have more levels of translation, such fall back can be worse.
**Definition**: Translation Lookaside Buffer, a special cache within MMU that accelerates address translation.
This is aligned with time and space locality as mentioned before. Memory mapping is page-aligned, so the value within a same page will be accessed more often, by keeping their page information in TLB, we can save a lot of translation cost.
### TLB Lookup
A TLB lookup goes through the entire TLB table. TLBs are often set-associative to reduce the comparison, which means the entry in TLB is not unique, we may have multiple entry to save the time of scanning through the whole buffer.
The cost of address translation with TLB can be calculated like this:
Cost(address translation) = Cost(TLB Lookup) + Cost(full translation)$\times$P(miss)
### TLB Miss
Caused by:
- Page not accessed before
- Page evicted due to limited TLB size
- Page mapping conflict due to association
- Other processes update the **page table**
### TLB Performance
Besides the formula about TLB performance, there are some tricks that can improve TLB performance.
#### Superpage
**Definition**: a set of contiguous pages in physical memory that map a contiguous regions of virtual memory, where the pages are aligned so that they share the same high-order (superpage) address. We can increase TLB hit ratio (a bigger target), but we will lose some fine-grained control, and some problem like inner fragment becomes worse.
#### TLB Prefetching
**Definition**: Prefetching page table entries into TLB before it's actually used. This is a common technique in speed up, and there are many kinds of prefetching like:
- Sequential: spatial locality
- Strided: for array-based computation
- Correlated: learn from the history, very smart but may have predict overhead

CPU also has instruction prefetching to improve CPU pipeline.

### TLB Consistency
**Definition**: consistency is a common issue in cache, which means the cache must be the same as the original data whenever the entries are modified.
- Process Context Switch: flush the TLB when there is a context switch, however modern OS can use tagged TLB, use some extra bits to distinguish page mappings from different processes or address spaces.
- Permission Reduction: discard the whole TLB, modern ones support the removal of particular TLB entries.
- TLB shootdown: on multi-processor, any processor changing their page table will need to flush other processor's TLBs as well.

## Memory Cache
**Block** is the minimal unit of caching. Often larger than 1 word or byte to exploit the spatial locality, modern Intel processors use 64 Bytes.
### Cache Lookup
#### Fully Associative
**Definition**: each address can be stored anywhere in the cache table.
So we need to compare the cache tag on each cache line. For example, if our block size is 32 Bytes, then we need 5 bits to select the Byte. In 32 bit case, the reset 27 bits will be used to compare.
The drawback is performance degrades with larger cache, because there are more tags to be compared.
#### Direct Mapped
**Definition**: map to one specific cache line through a Hash function. Use the tag as a veryfication.
For example, we still have 32 Byte block size, and the total size of cache is 1 KB, then we need 32 cache line (easy math), then given address `100000` and hash function `addr & 1111100000`, extracting the last 6 to 10 bits, which skips the byte select bits. We can get the hashed value is 1, so we go to cache line number 1 and check the tag stored there, if matches then get the value.
#### Set Associative
**Definition**: N-way set associative means we have N direct mapped caches operates in parallel, so we have N entries per cache index.
It is kind of lie between Fully Associative and Direct Mapped. If we have 2-way set, then with the same example in Direct Mapped, we only need 4 bits to choose the cache line, but we will get 2 entries now. So we add 1 bit to the cache tag. If we have 4-way set, then add 2 bit.
So if we have 1-way set, it would just be Direct Mapped. And if we have 32-way set, then it would just be Fully Associative.
### Cache Replacement
Direct Mapped only has one possibility, so it does not has such concern, but Set or Fully Associative needs to think about how to decide which cached value to evict.
Random is simple and has no overhead, sometimes simple is good.
First-In-First-Out (FIFO) could lead to some worst case in typical workloads like scan through small size array repetitively.
Least Recently Used (LRU) kind of smart, learning from the past, evict the least recently used.
Also we have Least Frequently Used (LFU).
### Further Knowledge
#### Page Coloring
When the page offset does not match with Set index and Line Offset, the set number of two different page may be the same, so the value with same page offset will frequently collide with each other. So OS invented Page Coloring the distinguish them with extra information.
# Lecture 8 Demand Paging
**Scenario**: modern programs require a lot of physical memory, but they don't use all their memory all of the time.
**Solution**: use main memory as cache for disk, "lazy" memory allocation.
## Memory-mapped Files
**Definition**: memory-mapped files is a segment of virtual memory that has been assigned a direct byte-for-byte correlation with some portion of a file or file-like source.
For example, you can open a file first, but since we are doing lazy allocation, so it is not fully in the memory now, then we can call `mmap` on it to get a pointer, and do some modification with it. This provides a illusion that we have the whole file in our memory, while we are actually processing a block (page) of data.
## The Dirty Bit & Page Eviction Policy
**Definition**: it is used to determine whether a page has been modified.
The page eviction policy becomes more important now. Because the cost of being wrong is high, we probably need to flush the data into disk, we need to interact with disk and that is slow.
FIFO, MIN, RANDOM, LRU, etc. LRU seems to be the best among them, let's explain how to implement it.
We can use a list to track used pages, on each use, we remove the page from list and insert it to the head, when evicting, we evict the tail page.
But this method still has problem, we need to know immediately when each page is used, so we can change its position in the list, but this involves many instructions for each hardware access. So it is inefficient.
In practice, people use **Approximate LRU**.
### Clocking algorithm
**Definition**: Implementation with the *use* bit. The use bit is initialized to 0 in page table, and set to 1 whenever there is a page access. When we need to evict a page, we look at the page under the hand, if the use bit is 1, we clear the bit and move the hand to next page, repeat until we find a 0 use bit, when find, evict it.
The running hand looks like a clock, right?
There is also a **variant** of clock algorithm, that is **N chance** algorithm: when we find a 0 use bit page, we give it a chance by increasing its counter instead of immediately eviction. When the counter reaches N, we do the eviction, and if its use bit is 1, we clear the use bit also clear the counter.
## Allocation of Page Frame
The allocation of memory (page frame) is also a philosophy, if we swap most part of a program then it may not be able to proceed further. Each process needs *minimum* number of pages.
Also the **Replacement Scopes** matters:
- Global replacement: process select replacement frames from set of all frames.
- Local replacement: each process select from their own set of allocated frames.

**Self-paging** means: each process is responsible for managing its own page faults and memory allocation, rather than relying on a global operating system-wide policy.
**Global page management**:
- each process/user is assigned its fair share of page frames using max min scheduling algorithm.
- when memory is full, the page eviction happens to the process with the most allocated memory, this avoids malicious attackers that want to starve resources.

## Summary
To support demand paging:
- CPU does memory management (MMU), a few bits in page table entry, etc.
- OS dose page table manipulation, eviction strategy and page fault handler, etc.
# Lecture 9 Scheduling
## Concept
Why we need scheduling? For multitasking and Concurrency. Only useful when there is not enough resources.
Preemption is the basic assumption for fine-grained scheduling. Such scheduling is handle mostly by OS.
The goal of scheduling policy is:
- Minimize Response Time
- Maximize Throughput
- Fairness

## Policies
### First-Come, First-Served
Just another name of First In First Out. We run any program until it is done.
It is simple, but short jobs get stuck behind long ones.
### Shortest Job First
Always schedule the job with the shortest remaining time. This leads to lowest average time. However it has two cons: starvation, the long job may always be surpassed by short jobs; hard to implement, hard to predict the remaining time of a task.
### Round Robin
Each process gets a small unit of CPU time, after quantum expires the process is preempted and added to the end of the ready queue.
It is fair and better for short jobs, but the frequent context-switching time adds up for long jobs.
Another disadvantage is cache state must be shared between all jobs with RR but can be devoted to each job with FIFO.
### Strict Priority Scheduling
**Definition**: strict priority scheduling has multiple priority queues, each for different priority. The policy always execute highest-priority runnable jobs to completion. Each queue can be processed in RR with some time-quantum.
The problem is starvation towards lower priority jobs, and deadlock with **priority inversion**, which means high priority jobs depend on low priority jobs that don't have chance to run.
To solve this we propose dynamic priority, period.
#### Earliest Deadline First
Each task is assigned a current priority based on how close the absolute deadline is, the scheduler always schedules the active task with the closest absolute deadline.
#### Fairness
We can give each queue some fraction of the CPU or we could increase priority of jobs that don't get service.
### Multi-level Feedback Queue
Assuming a mix of two kinds of workloads:
1. Interactive tasks: using CPU for a short time then yield for IO waiting. Low latency is critical.
2. CPU-intensive tasks: using CPU for a long period of time. The response time often does not matter much.

A naive version of MFQ is: maintaining many tasks queues with different priorities, and use following schedule rules:
1. If Priority(A) > Priority(B), A runs, B doesn't
2. If Priority(A) = Priority(B), A and B run in RR

The key here is how to set the priority. The intuition is if a job repeatedly relinquish the CPU while waiting for input from the keyboard, it shall be kept in high priority. Otherwise, if a job uses CPU intensively for long periods of time it priority shall be reduced.
Our solution is to assign a quota for each job at a given priority level, and reduces its priority once the quota is used up. So we have some more rules:
3. When a job enters the system, it is placed at the highest priority
4. If a job uses up its allotment while running, its priority is reduced
5. If a job gives up the CPU before the allotment is used up (like performing IO), it stays at the same priority level.

There are many issues with this naive version of MFQ. For example, if we have too many interactive jobs in the system, they will consume all CPU time, imagine they are running one after another in perfect order. Thus the long jobs will be starved.
One solution is priority boost:
6. After some time period S, move all the jobs in the system to the top-most queue

Or we can slice the time of queues.
But there are still many issues like counter measure, the user can put in a bunch of meaning less IO to keep job's priority to be high. Also how to parameterize the scheduler is still a question, how many queues should there be? How big should the time slice be per queue?
### Fair Share
**Definition**: this type of scheduler aims to guarantee that each job obtain a certain percentage of CPU time.
#### Lottery Scheduling
**Definition**: give each job some number of lottery tickets, on each time slice, randomly pick a winning ticket, on average the CPU time is proportional to the number of tickets given to each job.
Different tickets assign strategy determines different behavior of policy. To approximate shortest remaining time first, short running job gets more and long job gets fewer. To avoid starvation, every job gets at least one ticket.
#### Completely Fair Scheduler (CFS)
The goal of CFS is to fairly divide a CPU evenly among all competing processes.
CFS uses a counting-based technique known as **virtual runtime**. As each process runs, it accumulates virtual runtime, and when a scheduling decision occurs, CFS picks the process with the lowest virtual runtime to run next.
The scheduling interval is decided by `sched_latency`, it will be divided by number of process running, but if there are too many processes, the switch overhead will be too big, so we also have a `min_granularity`.
CFS also provides control over process priority, we can assign nice number to process, and larger nice means lower priority. The nice number is an index to some weight, with greater weight, the process can share bigger time slice.
#### Real Time Scheduling (RTS)
**Definition**: we might have some scenario that **predictability** is essential, maybe like some transaction system that requires transactions to be processed within limited time, so we have a deadline. Real time is about enforcing predictability, and does not equal fast computing.
Tasks can be preempted and they have deadlines and computation times, the closest task to deadline will have highest priority.
## Scheduling on Multi-processor
Recall the cache consistency is very important with multi-processor, and this leads to a disadvantage of centralized MFQ in multi-processor scenario, the huge overhead of cache consistency. MFQ is a data structure and we visit it in memory, if we use a shared one, it needs to be updated frequently and needs to synchronize across processors, that is huge overhead.
Also we will have MFQ lock contention. When running a process in MFQ, the corresponding processor will hold the lock of that process, and since this process has high priority, other processors will also try to grab the lock and run it, then they find the lock has already be taken, and waste some time.
The cache reuse, hit ratio will also be low for each processor. Imagine we have 4 processor with 1 cache block and 5 loop task.
Since then we propose **affinity scheduling**: a thread is always scheduled or you can say re-scheduled to the same processor.
## Evaluation about Scheduling Algorithm
- Deterministic modeling
- Queuing models
- Etc
# Lecture 10 Lock and Conditional Variable Design
**Atomic Operations**: an operation that always runs to completion or not at all. It cannot be stopped in the middle, and the states can not be modified in the middle. Atomic operations are the fundamental building blocks to make 
# Lecture 12 Readers/Writers and Deadlock
## Readers/Writers Lock
The motivation is suppose we have a database, and there are two kinds of operations: **read**, which never modify database and **write**, which read and modify database. Is using a single lock on the whole database a good idea? It is correct but not efficient. Because we can have many readers working at the same time, but for writers, only one can work at a time.
## Deadlock
### Four requirements for Deadlock
- Mutual exclusion
	- Only one thread at a time can use a resource.
- Hold and Wait
	- Thread holding at least one resource is waiting to acquire additional resources held by other threads.
- No preemption
	- Resources are released only voluntarily by the thread holding the resource, after thread is finished with it.
- Circular wait
	- There exist a set of waiting threads each waits the one after it.
### Preventing Deadlocks
1. No circular wait: careful design.
2. No hold-and-wait: use another lock to lock the locks, which may decrease concurrency.
3. No mutual exclusion: this may be very complicated, and need hardware support.
4. Smart scheduling: banking algorithm
### Banking Algorithm
Usually the request and assignment of resources are not determined at a single point, which means resources are taken/released over time. So we can't know order/amount of requests ahead of time, we can only assume some worst-case "max" resource needed by each process.
So we should state maximum resource needed in advance;
Only allow particular thread to proceed if available resources is greater than requested plus max remaining need resource.
