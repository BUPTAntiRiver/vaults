# General Cache Orgnization
Cache 的结构可以理解为一个大型二维数组，包含三个参数 $(S,E,B)$，$S$ 表示含有 $2^S$ 个 sets 即行数，$E$ 表示含有 $2^E$ 个 lines per set 即列数，$B$ 表示每个 cache block 中包含 $2^B$ 个字节的数据，此外还有 valid bit $v$ 表示这些数据是否有意义，例如电脑刚开机很多随机初始化的 cache 就没有意义，以及 tag 用来辅助搜索缓存。
## Cache Read 读缓存
Address of Word 读取缓存地址包含三个部分：
- tag
- set index
- block offset
在读取缓存的过程中，我们先根据 set index 定位目标在哪一个 set 中，然后搜索 line 匹配 tag 如果存在再检查 valid bit，一切正常表明读取命中。