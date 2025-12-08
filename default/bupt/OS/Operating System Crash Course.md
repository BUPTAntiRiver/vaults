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