# CPU-cache
## 基础知识
现在CPU多核技术，都有有几级缓存，新的CPU都会有三级缓存(L1, L2, L3)

<img src="https://github.com/lys861205/CPU-cache/blob/master/cache.architecture.png" width="500" heigth="400">

* L1缓存分成2种，指令缓存，数据缓存；L2/L3不分指令缓存和数据缓存
* L1和L2缓存在每个CPU核中，L3则是所有CPU共享
* L1,L2,L3 离CPU越近越小，速度也更快
* L1存取速度: **4个CPU时钟周期**
* L2存取速度：**11个CPU时钟周期**
* L3存取速度: **39个CPU时钟周期**
* RAM内存存取速度: **107个CPU时钟周期**

linux查看缓存大小(CPU0举例)
```
#一级缓存
cat /sys/devices/system/cpu/cpu0/cache/index0/size
32K
#一级缓存
cat /sys/devices/system/cpu/cpu0/cache/index1/size
32K
#二级缓存
cat /sys/devices/system/cpu/cpu0/cache/index2/size
1024K
#三级缓存
cat /sys/devices/system/cpu/cpu0/cache/index3/size
28160K
#cache line 大小
cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
64B
```
## 缓存命中
缓存基本上把后面的数据加载到离自己近的地方，加载数据单位是cache line；
比如：Cache Line 是最小单位(64Bytes)；所以先把Cache分成多个Cache Line；比如L1有32KB, 32KB/64B=512个Cache Line
Cache的放置策略决定了内存中的数据会拷贝到CPU Cache中的哪个位置上，需要有一种地址关联算法，能够让内存中的数据可以被
映射到Cache中来；基本方法如下
* 一种方法是，任何一个内存地址的数据可以被缓存在任何一个Cache Line里，这种方法是最灵活的，但是，如果我们要知道一个内存是否存在于Cache中，我们就需要进行O(n)复杂度的Cache遍历，这是很没有效率的
* 另一种方法，为了降低缓存搜索算法，我们需要使用像Hash Table这样的数据结构，最简单的hash table就是做“求模运算”，比如：我们的L1 Cache有512个Cache Line，那么，公式：（内存地址 mod 512）* 64 就可以直接找到所在的Cache地址的偏移了。但是，这样的方式需要我们的程序对内存地址的访问要非常地平均，不然冲突就会非常严重。这成了一种非常理想的情况了
* 为了避免上述的两种方案的问题，于是就要容忍一定的hash冲突，也就出现了 N-Way 关联。也就是把连续的N个Cache Line绑成一组，然后，先把找到相关的组，然后再在这个组内找到相关的Cache Line

## 缓存一致性
缓存的写操作有2种策略
* Write Back     写操作只在cache上，然后flush到内存上
* Write Through  写操作同时写到cache和内存上
为了提高写的性能，主流的CPU采用的是Write Back的策略，因为直接写内存太慢了
缓存一致性通过mesi协议更新，数据在缓存中有4种状态
Modified(已修改) Exclusive(独享) Shared(共享) Invalid(无效)

状态转移图

<img src="https://github.com/lys861205/CPU-cache/blob/master/MESI.png" width="500" heigth="400">

举例说明：
| 当前操作 | CPU0 | CPU1 | Memory | 说明 |
|---------|-------|------|-------|------|
|CPU0read(x)|x=1(E)||x=1|只有一个CPU有 x 变量，所以，状态是 Exclusive|
|CPU1 read(x)|x=1(S)|x=1(S)|x=1|有两个CPU都读取 x 变量，所以状态变成 Shared|
|CPU0 write(x)|x=9(M)|x=1(I)|x=1|变量改变，在CPU0中状态变成 Modified，在CPU1中状态变成 Invalid|
|变量 x 写回内存| x=9(M)|x=1(I)|x=9|目前的状态不变|
|CPU1read(x)|x=9(S)|x=9(S)|x=9|变量同步到所有的Cache中，状态回到Shared|


