# Array

如果稍微深入一点地学习过 C 语言那么这一块内容并不会有太多新知识。暂略。

# Data Alignment

当存储像 `Struct` 这样含有多种数据类型的数据结构时，会基于需要字节数最多的原始数据类型进行补齐。例如一个结构体含有 `char int char` 三个数据，那么在分配空间时，会为 `char` 也分配 4 个字节的空间，其中有 3 个空余字节，为了能够对齐 `int` 所需的 4 个字节。程序员可以自行调整顺序，来减少空余空间，例如按照 `int char char` 的顺序便只有 2 个空余字节。
