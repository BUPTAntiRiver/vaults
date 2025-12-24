# 初步理解过程
题目要求我们分析开始执行 `vector0` 函数后续的执行流程，即 OS (xv6) 做了哪些事、如何处理发生除 0 异常的用户程序。
那么首先，我们需要观察 `vector0` 函数本身。
```
globl vector0
# Divide Error Exception
vector0:
  # 为了在调用 trap 函数时构成一个完整的 struct trapframe 参数, 手动向内核栈顶压入 0 来充当 error code
  # 而对于含有 Exception Error Code 的异常来说, ISR 就不会手动压入 error code
  pushl $0
  # 压入trap number
  pushl $0
  jmp alltraps
```
这里我们看到，`vector0` 先压入了一个 error code 还有一个  trap number 然后 `jmp` 到了 `alltraps` 函数。其中 error code 为 0 因为当前 exception 没有 error code，起到一个占位符的作用，第二个压入的 0 是 trap number 告诉接下来要执行的函数我们遇到了什么 exception。
值得注意的是，此处注释里提到了 `trapframe` 结构体，我们来观察一下它的结构。
它定义在 `x86.h` 中：
```
struct trapframe {
  // registers as pushed by pusha
  uint edi;
  uint esi;
  uint ebp;
  uint oesp;      // useless & ignored
  uint ebx;
  uint edx;
  uint ecx;
  uint eax;

  // rest of trap frame
  ushort gs;
  ushort padding1;
  ushort fs;
  ushort padding2;
  ushort es;
  ushort padding3;
  ushort ds;
  ushort padding4;
  uint trapno;

  // below here defined by x86 hardware
  uint err;
  uint eip;
  ushort cs;
  ushort padding5;
  uint eflags;

  // below here only when crossing rings, such as from user to kernel
  uint esp;
  ushort ss;
  ushort padding6;
};
```
结构体在内存中的存储方式是代码从上到下，栈内从下到上，那么我们先来看代码的最下方，注释说明有很多 hardware 定义的变量，但是由于除 0 没有 error code 即此处的 `err` 所以我们手动压入了一个，接下来压入的就是再上方的 `trapno` 显然就是代表我们的 exception 的 trap number。

接着我们去看 `jmp alltraps` 到达的 `alltraps` 函数：
```
.globl alltraps
alltraps:
  # Build trap frame.
  pushl %ds
  pushl %es
  pushl %fs
  pushl %gs
  pushal

  # Set up data segments.
  movw $(SEG_KDATA<<3), %ax
  movw %ax, %ds
  movw %ax, %es

  # Call trap(tf), where tf=%esp
  # 这里压入栈顶指针, 栈顶恰好形成了struct trapframe
  # 可以思考一下CPU硬件、vector0、alltraps先后向内核栈压入值的顺序, 然后比较struct trapframe结构体的内部结构
  pushl %esp
  call trap
  addl $4, %esp
  # 此处函数没有ret
  # Return falls through to trapret...
```
一开始压入的四个 register 刚好对应 `trapframe` 中的内容，`pushal` 也呼应了注释，至于它们有什么用处，后续再看。
然后是三行 `movw` 指令，它起到的作用是把 `arg1` 的值原原本本得 copy 到 `arg2` 里面。所以现在 `ax`, `ds`, `es` 中的值都是 `$(SEG_KDATA<<3)` 具体这个值有什么用，我们也是后续再考量。
接下来压入了 `esp` 即栈顶指针，这导致了什么呢？设想一下，`esp` 本来指向哪里？指向我们最后一次压入后的栈地址，即之前构造的 `trapframe` 的第一个数据，那么此时的栈顶指针就相当于一个指向 `trapframe` 的指针，所以我们把它存储下来，方便后续访问 `trapframe`。
压入完毕后，我们调用 `trap` 函数。
还是先观察一下 `trap` 函数，由于它比较长就不复制了，首先看它的参数，是一个 `trapframe` 指针，也就是我们刚才存储的 `esp`。
一开始是一个判断，其中 `T_SYSCALL` 在 `traps.h` 中找到为 64，我们的 `trapno` 为 0，那么直接不用看了。
接下来是一个 `switch`，那我们就来看一下到底哪个 `case` 等于 0，同样，所有的 global constants 都可以在 `traps.h` 里找到。最后发现落入了 `default`。
```
default:
    if(myproc() == 0 || (tf->cs&3) == 0){
      // In kernel, it must be our mistake.
      cprintf("unexpected trap %d from cpu %d eip %x (cr2=0x%x)\n",
              tf->trapno, cpuid(), tf->eip, rcr2());
      panic("trap");
    }
    // In user space, assume process misbehaved.
    cprintf("pid %d %s: trap %d err %d on cpu %d "
            "eip 0x%x addr 0x%x--kill proc\n",
            myproc()->pid, myproc()->name, tf->trapno,
            tf->err, cpuid(), tf->eip, rcr2());
    myproc()->killed = 1;
  }
```
这里 `if` 调用了 `myproc()`，在 `proc.c` 找到，观察一下：
```
struct proc*
myproc(void) {
  struct cpu *c;
  struct proc *p;
  pushcli();
  c = mycpu();
  p = c->proc;
  popcli();
  return p;
}
```
发现它返回的是一个进程的指针，所以这里的条件判断就是判断是否进程不存在或者 `cs` 为 0 即处在 kernel mode 那么说明是系统出了问题。如果是在 user mode 出了问题，那么 `print` 报错，杀死进程。
```
if(myproc() && myproc()->killed && (tf->cs&3) == DPL_USER)
    exit();
```
接下来就是判断是否是一个用户态已被杀死的进程，然后强制退出，并且加以确认。
观察在 `prop.c` 中的 `exit` 函数，创建了一个指向当前进程的指针，然后关闭所有当前打开的文件，清除当前工作目录 `cwd`，让 `parent` 停止 `wait`，将当前进程所有子进程转移到初始进程 `initproc` 之下，最后将当前进程设置为 `zombie` 进入 `scheduler` 处理。
当 `scheduler` 处理完后，我们就重新回到了之前没有返回的 `alltraps` 继续执行：
```
addl $4, %esp
  # 此处函数没有ret
  # Return falls through to trapret...
.globl trapret
trapret:
  popal
  popl %gs
  popl %fs
  popl %es
  popl %ds
  addl $0x8, %esp  # trapno and errcode
  iret
```
这里让 `esp` 增加 4 相当于缩小了栈，这是栈一个比较反直觉的点，如果把 memory 视作一个竖着的长方形，且顶上为高地址，底下为低地址，那么**栈是从顶上往下长的**。所以增加栈顶地址相当于缩短了栈，也就是弹出了我们之前存储的指向 `trapframe` 的指针，接下来的 `pop` 则是弹出之间压入的寄存器值，相当于恢复 exception 处理之前的状态，最后又 `add` 了 8 也就是两个 32 位地址，弹出两个变量，即我们最开始压入的 `trapno` 和 `errcode`，一切又恢复如初，只是异常进程已经被处理掉了，CPU 将会继续执行后续指令。
# 总结
## 关键结构体及其作用
### trapframe
包含进入异常处理程序之前的所有信息，从而能够在完成处理之后恢复原来状态继续执行。
同时让处理程序根据 `trapno` 决定如何处理。
### proc
代表一个进程，也是处理对象。
## 关键函数
### 做了什么
#### alltraps
构建 `trapframe` 进入 `trap` 函数，从而能够处理异常进程。
#### trapret
让 CPU 恢复到处理之前的状态，清理使用的栈资源。
#### trap
判断是什么类型的 `trap` 然后调用相应函数处理。
#### exit
关闭当前进程，同时妥善处理与之相关联的所有进程，例如 parent 停止等待，children 转移到 `initproc` 之下，最后让 `scheduler` 处理变成 `zombie` 的当前进程。
### 流程关系
`vector0` -> `alltraps` -> `trap` -> `exit` -> `trapret`
# 简单补充
## 为什么 `esp` 增加 4 会弹出 1 个变量？
因为在内存地址中，地址每增加 1 相当于增加 1 个字节，也就是 8 个二进制位，而指针是用 4 个字节表示的，`trapno` 和 `errcode` 应该都是 `unsigned int` 也是 4 个字节。
## SEG_KDATA 这一段是在干什么？
```
  # Set up data segments.
  movw $(SEG_KDATA<<3), %ax
  movw %ax, %ds
  movw %ax, %es
```
首先，`SEG_KDATA` 是什么？这是由于内存地址的 segmentation 而出现的，对于一个 16 位的机器，如果永远直接读取地址，那么范围有多少呢？64 KB，非常小，但如果引入 segmentation，配合一个 4 位的 offset，就可以有 1 MB 的范围。
此处也是同样的道理，将 `SEG_KDATA` 左移三位后，最低的三位全部为 0，表明我们将使用 `Global Descriptor Table` 同时 `Requested Privilege Level` 为 0 即所谓的内核态。
`ds` 和 `es` 是两个 Segment 寄存器，分别表示 `Data Segement` 和 `Extra Segment`，表明我们使用系统的数据等等。