# P1
首先看下代码：
```
void *get_physaddr(void *virtualaddr) {
    unsigned long pdindex = (unsigned long)virtualaddr >> 22;
    unsigned long ptindex = (unsigned long)virtualaddr >> 12 & 0x03FF;

    unsigned long *pd = (unsigned long *)0xFFFFF000;
    // Here you need to check whether the PD entry is present.

    unsigned long *pt = ((unsigned long *)0xFFC00000) + (0x400 * pdindex);
    // Here you need to check whether the PT entry is present.

    return (void *)((pt[ptindex] & ~0xFFF) + ((unsigned long)virtualaddr & 0xFFF));
}
```
传入的参数是我们的虚拟地址，我们要得到的物理地址应当是处在的 page 地址加上偏置。
第 1 行：获得 page table 在 page directory 中的索引，10 位。
第 2 行：获得 page 在 page table 中的索引，10 位。
第 3 行：根据文档所说，习惯上 page directory 会把指向他自己的索引放在最后一个 PDE (page directory entry)，所以这里前 20 位全是 1，两次映射回自己，后续的检验就是为了证明这个指向自己的 entry 是有效的。
第 4 行：这里就是我们要构建一个全新的虚拟地址，前 10 位全 1，这样在 page directory 阶段会指向自己，接下来的 10 位是 `pdindex` 相当于我们把 page directory 当成 page table 进行查询，那么得到的就是指向原本虚拟地址所处在的 page table 的那个 page。最低 12 位为 0，相当于 offset 为 0，直接获得 page table 虚拟地址。
最后返回：前半部分是相当于给之前得到的 page table 虚拟地址加上 page 的索引作为偏置，然后解引用，得到 page 的物理地址，取前 20 位再加上虚拟地址后 12 位的偏置就得到了物理地址。
`pt[ptindex]` 相当于是对如下地址：|返回 page directory 自身的索引|`pdindex`|`ptindex`|解引用得到的值。
按照常规虚拟地址映射流程就是：
1. 查找在哪个 page table，此处我们回到了 page directory
2. 查找在哪个 page，这里我们处在 page directory，把它当成 page table 并且使用 `pdindex`，所以我们就得到了 page table
3. 添加偏置，得到物理地址，但这里我们把 page table 当成 page，偏置为 `ptindex` 于是得到的物理地址就是 page 的物理地址
相当于我们浪费了一次检索，从而得到了更高一层的物理地址。
# P2
讨论 page size 变成 8 KB 时的地址翻译。
首先 offset 从 12 位变成 13 位了。那么 entry 中 page 的地址变成 19 位，entry 可用的空间增加了。
entry 大小还是 4 个字节，一个 page 可以存放 2 K 个 entry，11 位，这也决定了 page table 的大小。
剩下 8 位构成了 page directory。
所以 page directory 1 KB，page table 8 KB