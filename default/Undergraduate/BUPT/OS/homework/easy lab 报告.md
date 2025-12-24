在本次实验中，主要需要理解每个函数的作用，负责的内容，从而避免重复完成某一功能或者缺失某一操作。
关于 context switch 也要注意顺序，切换之后再进行的操作都会在切换后的线程完成，所以需要把 switch 放到最后。
# 主要函数
## uthread_create
创建一个线程，初始化，关键在于往 context 中存放：
- `rip` 执行任务的入口 `_uthread_entry`
- `rsp` 栈指针
- `rdi` `rsi` `rdx` 传入 `_uthread_entry` 的参数
同时加入队列。此时状态为 INIT。
## `_uthread_entry`
执行线程任务的入口，因为 `rip` 指向它，所以当前线程运行时就会运行它，在其中我们需要运行 `thread_func` 并且在运行完成后，把线程状态设置为 STOP 返回主线程。
## schedule
负责不断调度子线程，从 `queue` 取出一个，只要状态不是 STOP 就可以 `uthread_resume` 恢复其上下文，即调度。
## uthread_resume
switch 到传入的线程，设置其状态为 RUNNING.
## uthread_yield
线程让出控制权，把自己状态设置为 SUSPEND，加入到队列中等待下次调用然后返回主线程进入 `schedule`。
此处我在实现时遇到了问题，会导致运行停滞，因为我没有暂存 current thread 直接把它设置为 main thread 导致 switch 的时候从 main switch 到 main。第一次修改把 current thread 的修改放在 switch 后面，但此时已经是 switch 后的 context，会直接执行 `schedule` 等再次返回之前的线程才会执行这行代码。最后采用了当前的实现。
# 思考
线程代码要注意当前所处上下文，以及正确的状态控制。
# challenges
## 1. 浮点数寄存器
一般浮点数操作涉及到的寄存器主要是 16 个 `xmm` 16 字节的寄存器，此外还有 `ymm` `zmm` 等，它们有着更大的容量，其之间的区别也主要在于宽度上的区别。
所以比起原来的实现我们需要在 `context` 结构中增加 `xmm` 的空间以及调整 `switch.S` 中的代码。由于 `xmm` 为 16 字节，所以我们需要调整对齐为 16，文档中提供的对齐方法很有效。
测试结果：
```
=== WITHOUT XMM register save/restore ===
Thread 1: Before yield - xmm0=100.123, xmm1=110.123, xmm2=120.123, xmm3=130.123
Thread 1:                xmm4=140.123, xmm5=150.123, xmm6=160.123, xmm7=170.123
Thread 2: Before yield - xmm0=200.123, xmm1=210.123, xmm2=220.123, xmm3=230.123
Thread 2:                xmm4=240.123, xmm5=250.123, xmm6=260.123, xmm7=270.123
Thread 1: After yield  - xmm0=200.123, xmm1=210.123, xmm2=220.123, xmm3=230.123
Thread 1:                xmm4=240.123, xmm5=250.123, xmm6=260.123, xmm7=270.123
Thread 1: xmm0 corrupted! Expected 100.123, got 200.123
Thread 1: xmm1 corrupted! Expected 110.123, got 210.123
Thread 1: xmm2 corrupted! Expected 120.123, got 220.123
Thread 1: xmm3 corrupted! Expected 130.123, got 230.123
Thread 1: xmm4 corrupted! Expected 140.123, got 240.123
Thread 1: xmm5 corrupted! Expected 150.123, got 250.123
Thread 1: xmm6 corrupted! Expected 160.123, got 260.123
Thread 1: xmm7 corrupted! Expected 170.123, got 270.123
Thread 1: ✗ FAILED! 8 XMM registers corrupted!
Thread 2: After yield  - xmm0=0.000, xmm1=0.000, xmm2=260.123, xmm3=270.123
Thread 2:                xmm4=240.123, xmm5=250.123, xmm6=260.123, xmm7=270.123
Thread 2: xmm0 corrupted! Expected 200.123, got 0.000
Thread 2: xmm1 corrupted! Expected 210.123, got 0.000
Thread 2: xmm2 corrupted! Expected 220.123, got 260.123
Thread 2: xmm3 corrupted! Expected 230.123, got 270.123
Thread 2: ✗ FAILED! 4 XMM registers corrupted!

=== WITH XMM register save/restore (FIXED) ===
Thread 1: Before yield - xmm0=100.123, xmm1=110.123, xmm2=120.123, xmm3=130.123
Thread 1:                xmm4=140.123, xmm5=150.123, xmm6=160.123, xmm7=170.123
Thread 2: Before yield - xmm0=200.123, xmm1=210.123, xmm2=220.123, xmm3=230.123
Thread 2:                xmm4=240.123, xmm5=250.123, xmm6=260.123, xmm7=270.123
Thread 1: After yield  - xmm0=100.123, xmm1=110.123, xmm2=120.123, xmm3=130.123
Thread 1:                xmm4=140.123, xmm5=150.123, xmm6=160.123, xmm7=170.123
Thread 1: ✓ PASSED! All XMM registers preserved
Thread 2: After yield  - xmm0=200.123, xmm1=210.123, xmm2=220.123, xmm3=230.123
Thread 2:                xmm4=240.123, xmm5=250.123, xmm6=260.123, xmm7=270.123
Thread 2: ✓ PASSED! All XMM registers preserved
```
## 2. 实现 m:n 模型
