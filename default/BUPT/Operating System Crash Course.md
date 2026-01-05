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

**Definition**: the execution of an application program with restricted rights. It differs from program, it is an instance of a program, like an object is an _instance_ of a class in Object Oriented Programming.

### Process Control Block

**Definition**: PCB is a data structure used by Linux to keep track of a process execution.

### Process In Memory

An executable mainly consist of BSS (Block Starting by Symbol), data and code regions. And the running process will have its own stack and heap. Uninitialized data are stored in bss region and initialized data are stored in data region.

## Dual Mode

### How to Virtualize?

The basic model of CPU virtualization: run one process for a little while, then run another one, and so forth.
This leads to 2 challenges:

1. **Performance**: how to virtualize without adding excessive overhead to the system?
2. **Control**: OS must take control whenever it wants; otherwise a process can run forever. OS must control how certain resources can be accessed by processes. We can just let OS take charge of every instruction of process, but this seems like we are running them on a Virtual Machine to test whether it is safe then actually run it, which is too slow.

So rather than always direct run or always go through OS, we perform the philosophy of Computer Science, absorbing the advantages. We design a method that **"sensitive" instructions go through OS and most instructions directly run on CPU**. The "sensitive" instructions are predefined by human, what are they?

### Limited Direct Execution

OS needs to check the instructions to be executed, in order to prevent detrimental behavior.
The following instructions are limited:

1. Restricted operations;
2. inter-process switching: voluntary switching like: system calls and involuntary switching like: timer interrupt (running for too long).

The next problem is how to implement?

### Dual Mode

User mode and kernel mode both are CPU status，which is controlled by OS.
The status is stored in `cs` register, 0 for kernel mode, 3 for user mode (use only 2 bits).

### Privileged Instructions

I/O read/write, context switch, changing privilege level, set system time... Any instructions that _could affect other processes_ are likely to be privileged.

### Memory Protection

Segmentation: base and bound.
Paging: every memory address a process sees is _"discontinuously"_ mapped to a physical address in memory.
The kernel code is usually placed at high address, but not low address, this is to prevent user accidentally wants to access kernel code (uninitialized pointers pointing to 0) and we can separate kernel code and user with the prefix of address easily.

# Lecture 3 Context Switch

## User to Kernel Mode Switch

### Exceptions

When processor encounter _unexpected_ condition.
e.g. divide-by-zero, perform privileged instructions, etc.

### Interrupts

_Asynchronous_ signal to the processor that some _external_ event has occurred.

### System calls

User processes _request_ the kernel to do some operations.

### Interrupt Vector Table

It stores the entries of different handlers for exceptions, interrupts and traps (all of three, the naming is confusing). There is a register pointing to the vector table which is stored in kernel memory, and each element in the vector points to a corresponding handler.
Nowadays we have something like Interrupt Descriptor Table (IDT), which tells CPU where the Interrupt Service Routines (ISR) are located.
_Entries_ are called "**Gates**" (the gate that separates user and kernel), its location is kept in **IDTR** (IDT register) and is loaded with **LIDT** assembly instruction.

### Interrupt Masking

_Disable interrupts_ and _enable interrupts_ are two privileged instructions. This means some interrupts can be masked out during processing some important things, but there are still some most important things. So, we have:

- **Maskable** interrupts: all software interrupts, all system calls, and partial hardware exceptions
- **Non-Maskable** interrupts: partial hardware exceptions

The masked interrupts are actually deferred but not ignored.

### Interrupt Stack

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
**Concurrency**: _dealing_ with a lot of work at once, one worker, multiple tasks.
**Parallel**: _doing_ a lot of work at once, multiple worker, multiple tasks.
Concurrency is about what does one core do, parallel is about what do multiple cores do.

## Thread Abstraction

**Definition**: thread is a single _execution sequence_ that represents a _separately schedulable task_. It is the **minimal scheduling unit in OS**.
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

To delete a thread, we can remove it from the ready list, so that it will never run again, we also need to free the per-thread state allocated for the thread. But how to free the space? We can't let the thread itself to do so, its just like you need to kill yourself then you also need to say "Hey, I'm gonna quit!", without a stack, the thread can do nothing. So, we add the deleted threads to a _finished_ threads list, their space will be freed by _other_ threads.

# Lecture 6 Address Translation

## Concept

As the name suggests, address translation translate from virtual memory address to physical memory address. If the physical address is valid, it will access it and work, if not valid, throw an exception.
The **GOAL** and motivation of address translation: memory protection, memory sharing, flexible memory placement, sparse addresses, runtime lookup efficiency, compact translation tables, portability, etc.

## Segmentation

**Simplest Approach**: _base_ and _bounds_ registers, every memory access is checked on those registers. We will need a segment table to record these bases, bounds and read write permissions.
However, in reality, we find the usage of physical memory is fragmented, there are a lot of "holes", this is because process comes and goes, not every process can fit in spare space. Many methods are designed to enable fine-grained allocation.
In real mode (for anyone who forgets about real mode, it is an older CPU mode, typically uses 16 bits address, don't mix it with kernel mode and user mode! They are OS modes!), there is no segment table, we shift left the segment register value and add offset value to it.
In protected mode (modern CPU mode), the segment table is called global descriptor table (GDT) or local descriptor table (LDT). Linear address = base address + offset.
Segmentation is easy to implement and quite straight forward, but the principle downside is: overhead of managing a large number of variable size and dynamically growing memory segments, it would be easy if you are satisfied with a bad strategy. And that will lead to _external fragmentation_, free space becomes non-contiguous. Compacting the memory will be slow, and even more complex if the segments can grow.

## Paging

**Definition**: allocating memory in _fixed-sized_ chunks called _page frames_. We have a _page table_ stores for each process whose entries contain pointers to the page frames.
Page table is more compact than segment table because it does not need to store bounds. And now the pages are scattered across physical memory regions, yet with each page, the memory access is still contiguous. The memory allocation becomes very simple, we only need to find a page frame.

### How does it work?

Since we have fixed page size, suppose we have a 4 KB page size (the most common one), then we let the lowest 12 bits of our virtual address be the offset, and use the first 20 bits (suppose we have 32 bits address) to locate the frame id in the page table. Then we combine the frame id and offset to get the physical address. This is also the only way to get physical address: combine frame id in page table with offset. This will be mentioned again later.

### How large is the page table?

Each entry in the page table takes 32 bits, that is 4 bytes and we have $2^{20}$ entries, that will be 4 MB. Remember we have a page table for each process, so now open your task manager and you will find out that most processes only need around 0.1 MB! What a waste if I need 4 MB page table to support such a tiny process! And this is the main disadvantage of page table method.

### Multi-Level Paging

To solve such overhead, we introduce multi-level paging, **page directory**.
Separate the address into 3 parts: **page directory number** (10 bits), **page table number** (10 bits), **page offset** (12 bits). We use page directory number and page directory register `CR3` (tells you which directory to visit) to locate which entry in the page directory to visit, which is the address of page table, then we use page table number to locate the frame id. Finally combine frame id with offset to get physical address.
Note that page directory and page table only takes 4 KB now. So for processes that only needs small space, the cost of page directory and page table now reduces to 8 KB. The philosophy behind such improvement is to make page table finer-grained.
We can even extend such multi-level to 3 or more layers.

#### Interesting Question: how to find the physical address of page? Or page table?

When we dereferencing a pointer, we get the value it points to, when we print it, we get the virtual address. How to get its physical address? The physical address is just frame id + offset. The offset is the same as lower 12 bits of virtual address, so we only needs to get frame id, which is stored in page table with a pointer pointing to it. So we need to build a pointer that points to the same place. In page directory, there is a special entry that stores position pointing to itself, which is also the value of `CR3`, so we can set the first 10 bits to this offset, and next 10 bits to the original 10 bits. Then when processing this address, we will go from page directory to itself then treat it as page table and go to the real page table. Then we can just set the 12 bits offset as the page table offset to get the frame id. Combine it with real offset, we get the physical address.
This method can also be used to get physical address of page table, we set the page directory jump to itself for 2 times, then we set last 12 bits with page directory offset to get the page table address. Combine it with page table offset to get entry address.
Jump to self for 3 times to get page directory address, which is just `CR3`.

---

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

When the page offset does not match with Set index and Line Offset, the set number of two different page may be the same, so the value with same page offset will frequently collide with each other. So OS invented Page Coloring the distinguish them with extra information. The extra info here is actually just unused info. They color comes from the page address directly.
Page coloring is not truly coloring, it is a new cache line routing policy.

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

**Definition**: Implementation with the _use_ bit. The use bit is initialized to 0 in page table, and set to 1 whenever there is a page access. When we need to evict a page, we look at the page under the hand, if the use bit is 1, we clear the bit and move the hand to next page, repeat until we find a 0 use bit, when find, evict it.
The running hand looks like a clock, right?
There is also a **variant** of clock algorithm, that is **N chance** algorithm: when we find a 0 use bit page, we give it a chance by increasing its counter instead of immediately eviction. When the counter reaches N, we do the eviction, and if its use bit is 1, we clear the use bit also clear the counter.

## Allocation of Page Frame

The allocation of memory (page frame) is also a philosophy, if we swap most part of a program then it may not be able to proceed further. Each process needs _minimum_ number of pages.
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

The key here is **how to set the priority**. The intuition is if a job repeatedly relinquish the CPU while waiting for input from the keyboard, it shall be kept in high priority. Otherwise, if a job uses CPU intensively for long periods of time it priority shall be reduced.
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

The resources are shared in computer system, so there may be conflict between different threads or processes. We need a method to ensure correctness.
**Atomic Operations**: an operation that always runs to completion or not at all. It cannot be stopped in the middle, and the states can not be modified in the middle. Atomic operations are the fundamental building blocks to make different threads work together.
Let's clarify some definitions here:

- **Synchronization**: using atomic operations to ensure cooperation between threads. All synchronization involves waiting.
- **Mutual Exclusion**: ensuring that only one thread does a particular thing at a time.
- **Critical Section**: piece of code that only one thread can execute at once.
- **Lock**: prevents something from doing something. We lock before entering critical section or accessing shared data and unlock when leaving.

## Lock

Implementation:

- `lock.acquire()` wait until lock is free, then grab
- `lock.release()` unlock, wake up anyone waiting

## Conditional Variable

**Definition**: a queue of threads waiting for something _inside_ a critical section.
Implementation:

- `Wait(&lock)` atomically release lock and go to sleep, re-acquire lock later, before returning
- `Signal()` wake up one waiter, if any
- `Broadcast()` wake up all waiters

### Producer-Consumer Example

```c
class bounded_queue {
	Lock lock;
	CV itemAdded;
	CV itemRemoved;
	void insert(int item);
	int remove();
}

void bounded_queue::insert(int item) {
	lock.acquire();
	while (queue.full()) {
		itemRemoved.wait(&lock);
	}
	add_item(item);
	itemAdded.signal();
	lock.release();
}

void bounded_queue::remove() {
	lock.acquire();
	while (queue.empty()) {
		itemAdded.wait(&lock);
	}
	remove_item(item);
	itemRemoved.signal();
	lock.release();
}
```

Two key principles of using conditional variables:

- CV is always used with lock acquired
- CV is put in a while loop, we need to check again and again, if we are not using loop, when insert is waked but failed to grab the lock and then some other insertion made the queue full, then it will try to add item! With loop we can re-check the condition.

## Semaphores

**Definition**: a kind of generalized lock. A semaphore has a non-negative integer value and supports the following two operations:

- `P()`: an atomic operation that waits for the semaphore to become positive, then decrements it by 1. It works similar to `wait()`
- `V()`: an atomic operation that increments the semaphore by 1, waking up a waiting `P`, if we have any. This is some kind of `signal()`

### Two use examples

1. Mutual Exclusion (initial value = 1), also called "Binary Semaphore", can be used for mutual exclusion.
2. Scheduling Constraints (initial value = 0), allow some thread to wait for signal from another thread.

### Producer Consumer

Implement Producer Consumer with semaphore. We need 3 semaphore in total, one to count used slot, one to count available items, one to act as a binary mutex.

# Lecture 11 Lock and Conditional Variables Implementation

## How to Implement Locks?

### With Interrupts

A naive method would be disable interrupts when acquire locks and enable interrupts when release locks. But we cannot let user do this! If they acquire lock and do a infinite loop, they can take over the system.
A better implementation would be putting both disable and enable interrupts in acquire and release. Disable and enable interrupts acts like a lock for lock, they set up a critical section, we can modify data structures to maintain lock information between them.
However, interrupts are enabled and disabled during context switch and it becomes really time consuming to implement on multi-processor. We have the following alternative.

### Atomic Read-Modify-Write Instructions

These instructions read a value and write a new value atomically, and hardware is responsible for this, such method can be used on both uni-processor and multi-processor.
Some examples are:

- `test&set(&address)` return value at `address` and set value at `address` to 1
- `swap(&address, register)` swap register's value to value at `address`
- `compare&swap(&address, reg1, reg2)` if register 1 and value at `address` are the same, set that value to register 2 and return success, else return failure

#### Implementation with `test&set`

**Spin lock**: a simple solution.

```c
int value = 0;

Acquire() {
	while (test&set(value));
}

Release() {
	value = 0;
}
```

The key idea is that we keep an **only single** 0 in all the values, so who has the 0 can break the while loop and enter the critical section.
The disadvantage is **busy waiting**, thread consumes cycles while waiting. How to solve this? The first thing come into mind is probably **conditional variable**, which can sleep and be signaled. That will be introduced later, with `test&set` only, we can also minimize wait cost with some engineering effort.

## How to implement Conditional Variables?

Recap the operations: `Wait(&lock)`, `Signal()`, `Broadcast()`. Since when we are using CV we are inside critical section, the implementation is quite simple, check by yourself.
Key note: we are using while to check the condition, the reason why is discussed in previous sections.

# Lecture 12 Readers/Writers and Deadlock

## Readers/Writers Lock

The motivation is suppose we have a database, and there are two kinds of operations: **read**, which never modify database and **write**, which read and modify database. Is using a single lock on the whole database a good idea? It is correct but not efficient. Because we can have many readers working at the same time, but for writers, only one can work at a time.

## Deadlock

### Four requirements for Deadlock

- Mutual exclusion: only one thread at a time can use a resource.
- Hold and Wait: thread holding at least one resource is waiting to acquire additional resources held by other threads.
- No preemption: resources are released only voluntarily by the thread holding the resource, after thread is finished with it.
- Circular wait: there exist a set of waiting threads each waits the one after it.

### Preventing Deadlocks

1. No circular wait: careful design.
2. No hold-and-wait: use another lock to lock the locks, which may decrease concurrency.
3. No mutual exclusion: this may be very complicated, and need hardware support.
4. Smart scheduling: banking algorithm

### Banking Algorithm

Usually the request and assignment of resources are not determined at a single point, which means resources are taken/released over time. So we can't know order/amount of requests ahead of time, we can only assume some worst-case "max" resource needed by each process.
So we should state maximum resource needed in advance.
Only allow particular thread to proceed if available resources is greater than requested plus max remaining need resource.

# Lecture 13 IO devices and disk

## IO Devices

**Definition**: IO devices usually have two parts, an interface and in internal structure. Some complex IO devices may have their own CPU and memory.
The interface usually has 3 registers:

- **status** register
- **command** register
- **data** register

To check the status, we may use polling, but that is too expensive, so we usually use interrupts.

### Direct Memory Access

DMA acts like a bridge between CPU and Disk, it is a worker listen to CPU instructions and transfer data from CPU to disk, so that CPU can spend its time on more important jobs. When DMA is complete, it will send a interrupt to notify CPU.

### IO methods

Two complementary ways for CPU to access IO devices:

- Memory-mapped IO (MMIO): let memory and devices share the physical address space. This is most widely adopted and they share a address bus.
- Port-mapped IO (PMIO), or isolated IO: use specialized instructions to read and write IO devices.

## Storage Devices

OS is deeply connected with hardware so we also need to learn about them.
Introduce two names here: magnetic disks and flash memory.

### Magnetic Disk

#### Definition

Made up of:

- **Sector**: the unit of transfer
- **Track**: rings of sectors
- **Cylinder**: stacked tracks, all the tracks under the head at a give point on all surfaces
- **Head**: attached to movable arms to read data, we have 2 per **platter** for each surfaces

Storage capacity = head count $\times$ cylinder count $\times$ sector count $\times$ sector size (usually 512 bytes)
We are not using track count here because head count $\times$ cylinder count is just track count!

#### Read and Write

We have three stages in this process:

- **Seek time**: to find position the head/arm over the proper track
- **Rotational latency**: wait for desired sector to rotate under r/w head
- **Transfer time**: transfer a block of bits under r/w head

The total latency also needs to add software time, that is queuing time and controller time, so in total it is: Disk Latency = Queuing Time + Controller Time + Seek Time + Rotational Latency + Transfer Time
The transfer time is impossible to reduce with same device, queuing time and controller time are decided by software, so the key to use disk **effectively** (especially for file systems) is to _minimize seek time and rotational latency_.

#### Intelligence in the Controller

When there is bad sectors (broken) in the disk, controller provides an illusion like it is totally fine (just like what OS do!), and re-map it to a fine sector on the same surface. Sometimes in order to keep sequential behavior (when there is a bad sector in the middle), controller may re-map all sectors to a new place.

### Solid State Disk

#### Read

There is no seek or rotational latency in SSD, the latency will be Queuing Time + Controller Time + Transfer Time. Also it has the highest bandwidth no matter we are doing Sequential or Random reads.

#### Write

Writing is complex, we can only write to empty pages in a block. And we can erase data to provide new spare spaces. The controller will maintain pool of empty blocks by coalescing used pages, also reserve some percent of capacity in order to do some optimization.
**One rule to keep in mind**: write is 10 times slower than read, erase is 10 times slower than write.

#### Summary

SSD has:

- Lower latency and higher throughput
- No moving parts (very light weight, low power, silent, very shock insensitive)
- Read at memory speeds (limited by controller and IO bus)

The disadvantages are:

- Smaller storage and expensive price, however these two cons are reduced greatly in the latest products, no longer true!
- Asymmetric block write performance: read/erase/write page, controller garbage collection (GC) algorithms have major effect on performance
- Limited drive lifetime. Average failure rate is 6 years, expectancy is 9 to 11 years.

These are changing rapidly! SSD is the main stream now, all disk enterprise are working on it!

## Disk Scheduling

Two not that good methods, the methods we are talking about are all based on Magnetic Disks

- FIFO, may lead to long seek
- SSTF, shortest seek time first, may lead to starvation

The more common one is SCAN, implementing with an Elevator Algorithm: take the closest request in a fixed direction of travel (reversed at the end), in this case we have no starvation and SSFT like flavor.
Another method is called C-SCAN (C stands for circular), which only goes in one direction, skips any requests on the way back. It is fairer than SCAN, not biased towards pages in the middle.

## A simple read() life cycle

1. A process issues a system call `read()`
2. OS moves the calling thread to a wait queue
3. OS uses memory-mapped IO to tell the disk to read the requested data and set up the DMA so the disk can place the data in kernel's memory
4. Disk reads data and DMA it into main memory
5. Disk triggers an interrupt
6. OS's interrupt handler copies the data from the kernel's buffer into the process's address space
7. OS moves the thread to ready state
8. The thread is scheduled on CPU, and return from the `read()`

## File System Abstraction

**Definition**: Layer of OS that transforms block interface of disk into Files, Directories, etc.
**Components**:

- **Naming**: Interface to find files by name, not by blocks
- **Disk Management**: Collecting disk blocks into files
- **Protection**: Layers to keep data secure
- **Reliability/Durability**: Keeping of files durable despite crashes, media failures, attacks, etc.

### Disk Management Policies

Basic entities on a disk:

- **File**: user-visible group of blocks arranged sequentially in logical space
- **Directory**: user-visible index mapping names to files

Access disk as linear array of sectors. Two options:

- Identify sectors as vectors \[cylinder, surface, sector\], sort in cylinder major order. This is usually used in BIOS, but not in OS anymore.
- **Logical Block Addressing** (LBA), every sector has a integer address from zero up to max number of sectors.
- Controller translate from address to physical position, in first case, BIOS needs to handle bad sectors, in second case, hardware shields OS from disk structure.

We also need ways to **track free disk blocks**. Naive methods like linking them together would be too slow now, we can use **bitmap** to represent free space on disk.
We need way to structure files: **File Header**. Which:

- Track which blocks belong to which offset within the logical file structure
- Optimize placement of file's disk blocks to match access and usage patterns

### File

**Definition**: named permanent storage.
**Contains**:

- Data: blocks on disk.
- Metadata (Attributes): owner, size, last open time, access rights, etc.

### Directory

**Definition**: basically a **hierarchical** structure. Each directory is a collection of **files** and **directories**, it has a name and attributes.
Hard links make it a **directed acyclic graph**, rather than a tree. Hard links are link to other files maybe already under other directory. Soft links (alias) are another name of en entry.
In shell, we have some convention for directory:

- Root directory: `/`
- Home directory: `~/dir/file.c`
- Absolute path: `/home/dir/file.c`
- Relative path: `file.c`

**Volume**: a collection of physical storage resources that form a **logical** storage device. Could be a part or many physical devices.
**Mount**: an operation that creates a mapping from some path in the existing file system to the root directory of the mounted volume's file system.

# Lecture 14 File System Design

## Components of File System

`Open` performs a **Name Resolution**, it translate path name into a "file number", creates a file descriptor in PCB within kernel, returns a "handle" (another integer) to user process.
`Read`, `Write`, `Seek`, `Sync` operates on the handle, then mapped to the file descriptor and to blocks.

### inode

**Definition**: an inode is a data structure on a file system on Linux and other Unix-like operating systems that store all the information about a file except its name and its actual data.
Q: Does each file has exactly one corresponding inode?
A: True for most traditional Unix-like file systems, but not true with hard links (covered later).
When a file is copied, a new inode is created. When a file is moved, nothing changed.
The number of inode is configurable in many file systems, depends on how many bits does the inode number have.

### In Memory File System Structures

`open` system call:

- Resolve file name, find file control block (inode)
- Makes entries in per-process and system-wide tables
- Return index (called "file handle") in open file table

`read` / `write` system call:

- Use file handle to locate inode
- Perform appropriate reads or writes

## Directory

### Structure

Directory is treated as a file with a list of (file name, file number) mappings.
The file number of the root directory is agreed ahead of time, in many Unix file systems, it is 2.
Modern file systems organize directory's contents as a tree, typically B+ tree, just like database systems!

### Hard Link

`ln` command, link: it creates another name in the directory you are creating the link to, and refers it to the same inode number of the original file. Os will maintain a reference count for each inode. When we create file 1 and link file 2 to it, even we remove file 1, we can still access data in that inode through file 2 because the reference count still has 1 now and is greater than 0. Usually used for version control.

### Soft Link

`ln -s` command, soft (symbolic) link: a special type of file whose contents are the path name of the linked-to file. The size of soft link file changes as they are linked to files with different names. Soft link does not add reference count. Usually used for shortcuts.

## Files

How do we locate storage blocks based on file number? It is different across file systems.

### FAT (File Allocation Table)

#### Features

FAT is a linked list as a one to one map with blocks.
Each entry in FAT contains a pointer to the next FAT entry of the same file, or a special end of file value.
FAT is stored on disk, on boot cache in memory. It also has a back up copy on disk.
To free space in FAT, we can just set the value of FAT list to 0. If we want to find a free space, we scan through the list.
To improve IO performance, locality is important, so when placing the files, we want to make sure it is stored in sequential blocks, but this is just like segmentation, the fragment will keep increasing.

#### Issues

Poor locality: there will be fragmentation.
Poor random access: scan through the list.
No support for hard link, because FAT does not use inodes.
File metadata stored in directory entries, therefore being limited: only has file name, size, creation time, but cannot specify the file's owner or group.
Limitations on volume and file size.

### Unix File Systems (FFS)

#### File Attributes

We have multi-level index in FFS, which can be represented as fixed, asymmetric tree. The entries are an inode array. In each inode, we have:

- File Metadata: 9 control bits (user, group, owner, read, write, execute)
- 12 Direct Pointers, each point to a 4 KB block, so each inode is sufficient for files up to 48 KB.
- Indirect Pointers, they point to a disk block containing only pointers. So a indirect pointer points to a 4 KB block that holds 1024 pointers, that will be 4 MB, and double indirect pointer turns out to be 4 GB, triple to be 4 TB.

#### Features

**Tree Structure**. Each file is represented by a tree, which allows the file system to efficiently find any block of a file.
**High Degree**.
**Fixed structure**.
**Asymmetric**. Direct and indirect pointers make FFS fits both small and big files very well.
**Locality Heuristics**:

- Block group placement: FFS places data to optimize for the common case where a file’s data blocks, a file’s data and metadata, and different files from the same directory are accessed together.
- Reserved space: reserved for locality optimization even when the disk is full, about 10%.

#### Free Space Management

FFS allocates a bitmap with one bit per storage block. The i-th bit in the bit map indicates whether the i-th block is free or in use.

#### Summary

**PROS**:

- Efficient storage for both large and small files
- Locality for both large and small files
- Locality for metadata and data
- No de-fragmentation necessity

**CONS**:

- Inefficient for tiny files (a 1 byte file requires both an inode and a data block)
- Inefficient encoding when file is mostly contiguous on disk
- Need to reserve around 10% free space to prevent fragmentation (kind of waste)

### NTFS (New Technology File System)

Almost everything in NTFS is a sequence of attribute, value pairs, for metadata and data. It mixes direct and indirect freely.

#### Master File Table

NTFS has a **Master File Table**, which is a database with flexible 1 KB entries for metadata/data. It has variable-sized attribute records, can be extended with variable depth tree.
For _small_ files, the MFT record contains default info and file name, then, it just store that file inside the record.
For _medium_ files, the MFT record stores start and length pairs to denote where are the data extents.
For _large or fragment_ files, the MFT record stores start length data info and other records info in attribute list too. Because we can store all the data info pairs inside one record.
For even rarer case _huge or badly fragment_ files, we can make the attribute list become non-resident too! (having extents) This is possible because everything in NTFS is described with MFT.

#### Locality Heuristics

The system tries to place a newly allocated file in the smallest free region that is large enough to hold it. NTFS also specifies the expected size of a file at creation time, which helps to plan space use. It also uses some reserved space to avoid fragmentation (about 12.5%).

## Virtual File Systems (VFS)

How to make different file systems work together? This need emerges as we have more and more kinds of devices and networked file system.
VFS is like java abstract Object, all file systems can implement it.

### Core VFS Abstractions

**Super Block**: file system global data
**Inode** (index node): metadata for one file
**Dentry** (direct entry): name to inode mapping
**File Object**: pointer to dentry and cursor (file offset)
Super block and inodes are implemented by file system developer.

## Summary

File System:

- Transform blocks into Files and Directories
- Optimize for size, access and usage patterns
- Maximize sequential access, allow efficient random access
- Projects the OS protection and security regime (UGO vs ACL)

File defined by header, called "inode" (index node).
Naming: translating from user-visible names to actual system resources
Multi-level Indexed Scheme: indirect pointer or extent.

# Lecture 15 Reliable File Systems

Threats to File System Reliability:

- Operation interruption
- Loss of stored data

What a Reliable File System does?

- "All or Nothing"
- Quite similar to critical section problem in concurrency

## Transactions for Atomic Updates

### Careful Ordering A Not That Good One

We can sequence operations in a specific order, so that sequence can be interrupted safely.
Enable post-crash recovery, read data structures to see if there were any operations in progress, clean up or finish as needed.
This is the approach taken by FAT and FFS.
There are some **issues**:

- Complex reasoning, there are so many possible operations and failures
- Slow updates, file systems are forced to insert sync operations barriers between dependent operations.
- Extremely slow recovery, need to scan all of its disks for inconsistent metadata structures

### Transactions Itself

Use _transactions_ for atomic updates, it is like a group of atomic updates. The system can be either before transaction or after transaction, not allowed to be in the middle of the transaction.
**Definition**: an **atomic sequence** of actions (read/write) on storage system (or database, i first learn this pattern from database). That takes the system from one **consistent state** to another.

#### Typical Structure

**Begin** a transaction: get transaction id.
Do a bunch of updates, if any fail along the way, **roll back**, or if any conflicts with other transactions **roll back**.
**Commit** the transaction.

#### Key Properties

- **Atomic**: all actions in the transaction happen, or none happen
- **Consistency**: transactions maintain data integrity, e.g. balance cannot be negative, or cannot reschedule meeting on February 30
- **Isolation**: execution of one transaction is isolated from that of all others; no problems from concurrency
- **Durability**: if a transaction commits, its effects persist despite crashes

#### Logging

**Definition**: instead of modifying data structure on disk directly, write changes to a log, once all changes are in log, it is safe to apply changes to data structures on disk. Log is **append only**.
If we crash **during logging**, the recovery is also faster, we can just look into the log, detect transactions with no commit, then discard log entries and disk remains unchanged.
Recover after **commit**, we can detect committed logs are redo them (may be scheduled later).

#### Detail

Repeated write-backs are OK, because they don't depend on previous data in the sector, however for operations like add value to data in sector won't be recoverable.
New requests need to consult the log first to ensure the data consistency, this can be alleviated by caching.

### Transaction File Systems

Two way to use transactions in file systems: journaling and logging.
**Journaling**: apply updates to the system's metadata via transactions.
**Logging**: apply both metadata and data in transactions.

## RAID

The storage devices itself may be broken some time, to keep the data still reliable under such cases, we have RAID (redundancy arrays of inexpensive disks). There are five levels of RAID initially (more now).

### RAID 1: Disk Mirroring/Shadowing

Each disk is fully duplicated onto its "shadow". This is the most expensive solution, because we need 100% capacity overhead. Also, we need to do the write twice, but read can have higher bandwidth.

### RAID 5+: High IO Rate Parity

Data are stripped across multiple disks, for example we have 5 disks and 4 data block, then the data are store across 0 to 3, disk 4 instead of storing actual data, it is responsible for storing the parity data, a recovery info constructed with X-OR operation over all previous 4 data block. So that once any one of them is down, we can X-OR the survived ones with parity to regain the lost data.
Also, we won't let one disk do all the parity job, each one store the parity info in turns. This is **rotating parity**, parity needs to be changed very often, so we split the workload.

# Lecture 16 Virtual Machine

Virtualization is a technology that allows for the creation of virtual versions of physical resources.
Virtual machine is a very hot topic in both academia and industry, it is an old idea, but becomes very successful with cloud computing. Now we have, Virtual Machine Monitor (VMM), also named Hypervisor.

## What is a VM?

We have already seen virtualization in OS: system calls, virtual memory, file system, etc. However a VMM virtualize an entire physical machine. The interface supported is hardware level (OS defines a higher level interface), VMM provides an illusion that software has the full control over the hardware, while actually VMM is under control.
This means that we can boot an operating system in a virtual machine; run multiple instances of an OS on same physical machine; run different OS simultaneously on the same machine.
VMM is like the OS for OS, it creates an illusion that the OS has its own private CPU and a large virtual memroy for each OS running on top of it. VMM is similar to OS because they both focus on virtualizing hardware, OS virtualize CPU and memory, VMM virtualize other hardware that supports OS.

## Why VM?

Machines today are very powerful, we want to multiplex them so that one machine can support the need of multiple workers. VM is also easier to migrate across machines, it is secure, portable, and can be used for emulation, etc.

## How does VM run?

VMM runs with privilege, OS in VM runs with lesser privilege (like user-level). Since we are going to run OS code in a VM directly on CPU, we need to make OS looks like a user-level process. We also want privilege instructions to trap.
We have some concepts here: _virtualization, simulation, emulation_.
Emulation aims at providing an environment of hardware or software. Simulation aims at mimicking certain process or object.

### Implementation

What needs to be virtualized? CPU, events (exceptions and interrupts), memory and IO devices.
This seems like we are just duplicating OS functionality in VMM, and indeed yes it is, but we are implementing a different abstraction. We used to face hardware interface, but now we have an OS interface.
What does hardware do for OS?

- _Privilege levels_: user and kernel.
- _Privileged instructions_: instructions available only in kernel mode.
- _Memory translation_: prevents user programs from accessing kernel data structures and aids in memory management.
- _Processor exceptions_: trap to the kernel on a privilege violation or other unexpected event.
- _Timer interrupts_: return control to the kernel on time expiration.
- _Device interrupts_: return control to the kernel to signal I/O completion.
- _Inter-processor interrupts_: cause another processor to return control to the kernel.
- _Interrupt masking_: prevents interrupts from being delivered at inopportune times.
- _System calls_: trap to the kernel to perform a privileged action on behalf of a user program.
- _Return from interrupt_: switch from kernel mode to user mode, to a specific location in a user process.

Quite a lot!

#### Virtualizing Privileged Instructions

OS in virtual machine can no longer execute privileged instructions. For those instructions that cause an _exception_, we trap into VMM, take care of business, return to OS in VM.
For those that do not cause an exception, different VMM has different methods. And hardware also has supported new CPU mode, instructions to do **trap and emulate**.
In real OS, when we try to run privileged instructions, we interrupt to hardware, it catches info and then transfer control to OS, finally OS handles and return. However in VM, we don't have such hardware to do the intermediate work for us. Instead we let VMM to do this. It catches interrupts and call corresponding handler installed in VM OS.

#### Virtualizing the CPU

VMM needs to multiplex VM on CPU, time slice the VM, each VM will slice its OS/application during its quantum. And use a typically relative simple scheduler: Round Robin, working-conserving (give unused quantum to other VM).

#### Virtualizing Events

VMM receives interrupts, exceptions.
Needs to vector to appropriate VM, like modify OS to use virtual interrupt register, event queue, or craft appropriate handler invocation, emulate event registers.

#### Virtualizing IO

OS in VM can no longer interact directly with IO devices.
**Xen**: modify OS to use low-level IO interface (hybrid)
**VMWare**: VMM supports generic devices (hosted)

#### Virtualizing Memory

Each OS thinks of physical memory as a linear array of pages and assign each page to itself or user processes. But VMM partitions memory among VM, so VMM needs to add an extra layer of virtualization makes "physical" memory a virtualization on top of what the VMM refers to as **machine memory**.
Now the work flow becomes like this: each OS maps virtual-to-physical addresses via its per-process page tables, then VMM maps the resulting physical address to underlying machine address via its per-OS page tables.
Hardware-managed TLB makes this difficult, when TLB misses, the hardware automatically walks the page tables in memory. As a result, VMM needs to control access by OS to page tables.
**Xen** implementation: page table work the same as before, but OS is constrained to only map to the physical pages it owns.

#### Example of Address Translation

In a 4-level page table in VM, each translation requires up to 24 memory accesses. Because first, we need find the next level address for guest address 4 times, and each time, we need to go to host, and the host also has a 4-level page table, so 4 time for each phase. In the end, we also need 1 more memory access to pass the result back to guest address, so that will be 20 in total. After we get the guest physical address, we still need to do translation in host, that will cost another 4 memory access, so add up to 24 times.
For 5-level page table, it should be 35 times.

#### Shadow Page Tables

VMM creates and manage page tables that map virtual pages directly to machine pages. These VMM page tables are the shadow page tables, they are loaded into the MMU on a context switch.
VMM needs to keep its shadow page tables up-to-date. When guest kernel changes its page table, it will trap into VMM because OS page tables are set read only. Then VMM applies write to shadow page table and OS table. This cause some more overhead.
Nowadays hardware support for shadow page table directly. The hardware can be set up with 2 page tables. The hardware can translate the guest virtual address to guest physical address then to host machine address directly.
This simplifies the VM implementation and make it faster, but adds overhead to the synchronization between shadow page table and guest page table.

#### Memory Allocation

VMM tends to have simple hardware memory allocation policy, just allocate static size with no dynamic resizing based on workload. And guest OS are not designed to handle changes in physical memory and often **no swap to disk**.
There is more advanced approach, like overcommit with balloon driver, which means there is a balloon driver in the OS to collect unused available resources, so that VMM can assign them to other VM. When memory pressure decreases, the balloon driver shrinks and give out more memory in VM OS.
Further optimization can be share identical physical pages like some all zero pages across VM to save memory space.

#### Hardware Support

Hardware company like intel and AMD added more and more features to work with virtual machine better.

## Usage

### Java Virtual Machine (JVM)

Java programs run on top of JVM, that is why it is ubiquitous and runs slower than native C code. It supports garbage collection and just-in-time compile.

### VM v.s. Container

The difference between VM and container is that suppose we have a hardware, VM will has a VMM, which controls multiple VM with different guest OS running on them. While container has a host OS kernel, in each container they runs different apps and libraries, which means they have same OS!
