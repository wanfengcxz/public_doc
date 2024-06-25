to do

RMSNorm

RoPE





在不损失精度的情况下，加速attention

# Motivation

注意力机制的时间/空间复杂度是O(n^2*d)，增大上下文长度会使得计算时间急剧增大。

多年来，GPU的计算能力（FLOPS）的增长速度比增加内存吞吐量（TB/s）更快。

如果没有数据需要处理，那么额外的TFLOPS的计算能力是没有意义的。这两者需要紧密配合，但自从硬件失去了这种平衡，我们必须通过软件来进行补偿。因此需要算法能够感知IO(IO-aware)。根据计算和内存访问的比例，一个操作可以分为：

- 计算受限型(比如矩阵乘法)
- 内存受限型(比如activation, dropout, masking等**element-wise操作**和softmax, layer norm, sum等**reduction操作**。)

element-wise的操作在计算时**只依赖**当前值，比如把每个元素都乘以2。

reduction**依赖所有的值**(比如整个矩阵或者矩阵的行)，比如softmax。

而Attention的计算是内存受限的，因为它的大部分操作都是element-wise的。我们可以看一下论文中测试的实际Attention的运行情况：

![image-20240624155414704](flash_attention.assets/image-20240624155414704.png)

在左侧栏上看到，masking、softmax和dropout操作占用了大部分时间，尽管大部分FLOPS都用在矩阵乘法中，但它们的时间却花的不多。内存不是一个单一的构件，它在本质上是分层的，一般的规则是：内存速度越快，成本越高，容量越小。在计算机体系结构里，存储都是分层的：

![image-20240624155451123](flash_attention.assets/image-20240624155451123.png)

原始attention流程：

![image-20240624160008142](flash_attention.assets/image-20240624160008142.png)

可以看出，每次做计算时都需要把数据从HBM读入SRAM，计算完毕又将数据写回。因此最容易相对的优化方法就是避免这种来回的数据移动。这就是那些编译器优化中的kernel fusion。

![image-20240624161309525](flash_attention.assets/image-20240624161309525.png)

算子融合可以大大提升计算效率。

注意力分数矩阵S和P都是`NxN`的，即空间复杂度是O(N^2)，这正是FlashAttention需要解决的瓶颈，它将内存复杂度降低到O(N)。

FlashAttention基本上归结为两个主要思想：

- Tiling（在前向和后向传递中使用）- 简单讲就是将NxN的softmax/分数矩阵划分为块。
- 重新计算（仅在后向传递中使用 - 如果您熟悉activation/gradient checkpointing，这将很容易理解）。



# Computer Arch

flash attention涉及到软硬件协同设计。

# Online softmax

attention最核心的三部（忽略scaling和mask）：

![image-20240624104219975](flash_attention.assets/image-20240624104219975.png)

`X`为`NxN`，`Q`为`Nxd`

在fp16的条件下，如果`x>=11`， `e^x`就会溢出。导致训练出问题。

于是研究者提出**safe-softmax**，`xj`是`X`行向量中的最大值。

 ![image-20240624104600072](flash_attention.assets/image-20240624104600072.png)

计算流程如下：

![image-20240624104949574](flash_attention.assets/image-20240624104949574.png)

三次循环效率低下，于是做出改进：

![image-20240624111300208](flash_attention.assets/image-20240624111300208.png)

得到新的计算方式：

![image-20240624111407482](flash_attention.assets/image-20240624111407482.png)

# Flash Attention

![image-20240624152825149](flash_attention.assets/image-20240624152825149.png)



进一步地，在attention整体的流程中，可以进一步优化，变成一个循环。循环少了意味着访存次数就少了。

![image-20240624112351850](flash_attention.assets/image-20240624112351850.png)

![image-20240624112336046](flash_attention.assets/image-20240624112336046.png)

