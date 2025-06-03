## 一、异常控制流（Exceptional Control Flow, ECF）

### 为什么需要 ECF？

传统控制流机制（跳转、分支、函数调用/返回）无法处理系统状态的变化，如：

- 数据从磁盘/网络到达
- 除零错误
- 用户按下 Ctrl-C
- 系统定时器到期

### 四种 ECF 机制

1. **异常（Exceptions）**：硬件与内核共同处理
2. **上下文切换（Context Switching）**：定时器触发进程切换
3. **信号（Signals）**：用于进程间通信
4. **非局部跳转（setjmp/longjmp）**：由 C 库实现

---

## 二、异常（Exceptions）

### 定义

当特定事件（如除零、页错误）发生时，处理器转交控制权给操作系统内核。

### 分类

#### 1. 异步异常（Interrupts）

- 来自外部设备（如键盘、定时器）
- 返回下一条指令继续执行

#### 2. 同步异常

- 由指令执行引起，分为：
- **Trap（陷阱）**：如系统调用，返回下一条指令
- **Fault（故障）**：如页错误，可能恢复并重试当前指令
- **Abort（终止）**：如非法访问，程序直接终止

### 实现机制

- 每种异常有编号
- 异常编号用作索引，查表进入对应处理器函数（异常表）

---

## 三、进程（Processes）

### 定义

进程是程序的一个运行实例，具有：

- **逻辑控制流**：看似独占 CPU
- **私有地址空间**：通过虚拟内存实现

### 并发执行

- 多进程并发运行由上下文切换实现
- 单核 CPU：交替运行（多任务）
- 多核 CPU：并行运行多个进程

---

## 四、进程控制（Process Control）

### 进程状态

- **运行中**：正在或即将被调度执行
- **停止**：暂停执行，等待事件或信号
- **终止**：永久结束

### 常用系统调用

- `fork()`：创建子进程（返回两次）
- `execve()`：加载并执行新程序
- `exit()` / `_exit()`：终止进程
- `wait()` / `waitpid()`：等待并回收子进程资源

### fork 特点

- 子进程拥有父进程的副本（虚拟地址空间、打开文件等）
- 子进程 PID 与父不同
- 输出顺序不可预测，需要 `wait()` 等待同步

### 僵尸与孤儿进程

- **僵尸进程**：已退出但未被回收
- **孤儿进程**：父进程已终止，由 init（PID=1）接管

### 示例函数

- `getpid()`：获取当前进程 PID
- `getppid()`：获取父进程 PID

---

## 五、错误处理机制

### 错误返回

- 系统调用失败返回 -1
- 错误原因存储在全局变量 `errno`

### 错误处理方式

```c

if ((pid = fork()) < 0) {

    fprintf(stderr, "fork error: %s

", strerror(errno));

    exit(1);

}

```

### 错误处理函数封装

```c

void unix_error(char *msg) {

    fprintf(stderr, "%s: %s

", msg, strerror(errno));

    exit(1);

}



pid_t Fork(void) {

    pid_t pid;

    if ((pid = fork()) < 0)

        unix_error("Fork error");

    return pid;

}

```

---

## 六、进程图与执行顺序

### 进程图（Process Graph）

- 顶点：语句执行
- 边：表示“先发生”
- 任意合法的拓扑排序对应一种可能的执行顺序

### 多次 fork 示例

- 多个 fork 可能产生多个进程，输出组合繁多
- 嵌套或顺序调用 fork() 会带来不同的图结构和输出模式

---

## 七、子进程资源回收（Reaping）

- 子进程终止但父进程未调用 `wait()` 会形成僵尸进程
- 使用 `wait()` / `waitpid()` 收回资源
- 如果父进程退出，init 进程会自动回收其子进程

---

## 八、同步（wait / waitpid）

### wait()

```c

int wait(int *child_status)

```

- 阻塞当前进程，直到任一子进程结束
- 可使用宏 WIFEXITED、WEXITSTATUS 等判断退出信息

### waitpid()

```c

pid_t waitpid(pid_t pid, int *status, int options)

```

- 等待特定子进程结束
