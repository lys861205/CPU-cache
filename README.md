# CPU-cache
## 基础知识
现在CPU多核技术，都有有几级缓存，新的CPU都会有三级缓存(L1, L2, L3)

<img src="https://github.com/lys861205/CPU-cache/blob/master/cache.architecture.png" width="500" heigth="400">

* L1缓存分成2种，指令缓存，数据缓存；L2/L3不分指令缓存和数据缓存
* L1和L2缓存在每个CPU核中，L3则是所有CPU共享
* L1,L2,L3 离CPU越近越小，速度也更快
- L1存取速度: **4个CPU时钟周期**
- L2存取速度：**11个CPU时钟周期**
- L3存取速度: **39个CPU时钟周期**
- RAM内存存取速度: **107个CPU时钟周期**
