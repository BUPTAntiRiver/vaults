# 分块
首先我们观察一下 cache line 相关的数据，其中最关键的是 cache line size，因为当我们访问一个内存地址时，cache 会把一个 cache line size 大小的数据存入 cache 体现出 space locality，那么当我们访问矩阵中的一个 float 时，cache 会存 16 个进去。但这似乎并不是问题的关键，因为我们要利用整个 cache，所以我们选择矩阵的一个局部，把他们全部存储在cache中进行反复操作，以提高 hit rate，从而提高运行效率。
# 数据重排
当我们访问第二个矩阵时，常规做法会一直改变行号，对于二维数组，每一行是连续存储的，行与行之间隔着一整行的距离，所以按照行访问会很容易超出 cache 范围，所以我们可以先对二号矩阵转置，这样访问数据就是按照行来访问，提高 hit rate。
但我们也可以通过改变索引顺序来直接提高 hit rate，从 ijk 变成 ikj 每次对 result 的一整行进行更新：
```
for j in range(P):
	result_matrix[i][j] += m1[i][k] * m2[k][j]
```
# Loop unrolling
对于循环而言，我们希望能够在一次循环内进行更多次的操作，这样就可以减少对于循环变量的访问，从而提高 cache hit rate。对于 ijk 三层循环分别采用了不同的步长，j 的最大，因为我们主要在对 j 进行扫描。
```
for j in range(0, P, 16):
	# operations over j through j + 15
```
# 线程
对于多核处理器我们用多线程可以进行加速，每个线程负责若干行的任务。同时线程设计的数据修改行不重叠，也不需要 mutex 来保证正确性，非常方便。
# SSE3
SSE3 的本质是一个大寄存器，128位装两个双精度浮点数，从而能够同时操作两个数，实现速度的大幅度提升，由于我一开始在一些超参数（block size, unroll size）上的选择比较保守，没有完全利用机器的性能，所以比较早就使用进行优化了，提升较大。
```
# With SSE3
g++ main.cpp matrix.cpp multiply.cpp -std=c++1z -pthread -mfma -o main_sse -D N=1024 -D M=1024 -D P=1024 && ./main_sse
[LOG] N=1024, M=1024, P=1024
[LOG] JUDGE_TIME
[LOG] loop=0, time=277174.021000 us
[LOG] loop=1, time=252833.968000 us
[LOG] loop=2, time=261809.023000 us
[LOG] loop=3, time=274811.421000 us
[LOG] loop=4, time=317462.672000 us
[LOG] average time=276818.221000 us
[LOG] GFlops = 7.757740

# Without SSE3
g++ main.cpp matrix.cpp multiply_no_sse.cpp -std=c++1z -pthread -o main_no_sse -D N=1024 -D M=1024 -D P=1024 && ./main_no_sse
[LOG] N=1024, M=1024, P=1024
[LOG] JUDGE_TIME
[LOG] loop=0, time=488119.322000 us
[LOG] loop=1, time=444572.297000 us
[LOG] loop=2, time=452288.520000 us
[LOG] loop=3, time=430790.647000 us
[LOG] loop=4, time=446932.792000 us
[LOG] average time=452540.715600 us
[LOG] GFlops = 4.745393
```
# 多线程不如单线程
当线程数量过多时，线程 context switch 带来的开销可能使得程序性能下降，得不偿失。
# 内存缺页
首先在 warm up 阶段肯定会遇到 page fault 申请 page，然后如果 memory 满了 swap out 再 swap in 也会遇到 page fault 但这种情况对于当前问题还算较难遇见，因为使用的内存大小目前不超过 1GB。
此外我们还是用了 block 来保证同一段时间内操作的内存是一个 block 内部的，这就避免了 naive implementation 下经常出现 page fault 的问题（因为矩阵换行）。所以在我当前的实现下，page fault 的出现大大减少了。
# GPU
为什么 GPU 在处理矩阵计算时会比 CPU 快很多呢？因为 GPU 的特点就是**核心数非常多**但是它的 **cache 是大量共享的**！所以 GPU 更适合做**单调的大量的**任务，比如矩阵计算，而 CPU 每个核心有自己的 L
 和 L2 cache，灵活度更加高，能处理复杂任务。
# 遇到的问题
最大的问题似乎在于 block size 的选取上，一开始没有理解 block 的意义，选取了特别小的 block size 但事实上，只要 cache 里装得下应该用更大的，同时每个 core 都有自己的 cache 所以我们为了提高效率，应该尽量使用每个 core 的 L1 还有 L2 cache，由于 L1 太小了，所以我们主要考虑 L2，它在大小的速度间取得了比较好的平衡，最后选择了 128 by 128 by 512 的 block size。