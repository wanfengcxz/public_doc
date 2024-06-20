# GEMM

## Naive GEMM

![image-20240617194347081](GEMM.assets/image-20240617194347081.png)

**每个thread负责读取A矩阵的一行和B矩阵的一列，去计算C矩阵的一个元素**。则一共需要`M*N`个thread。

**矩阵A和矩阵B都存储在global memory，每个thread直接从global memory上进行读数**，完成计算：

- 为了计算出C中的某个元素，每个thread每次都需要从global memory上读取A矩阵的一行（K个元素），B矩阵的一列（K个元素），则每个thread从global memory上的读取次数为`2K`。
- C中共有`M*N`个thread，则为了计算出C，对global memory的总读取次数为`2KMN`。

这里及之后的分析中，我们不考虑把结果矩阵C写回global memory需要的次数，只考虑“读”。

Naive GEMM的代码见下（完整代码见 https://github.com/ifromeast/cuda_learning/blob/main/03_gemm/sgemm_naive.cu）：

- blockDim：`(32, 32)`，因为一个block内最多1024个thread
- gridDim：`(16, 16)`

```c++
// 将二维数组的行列索引转成一维数组的行列索引，这样可以更高效访问数据
// row, col：二维数组实际的行列索引，ld表示该数组实际的列数
// 例：二维数组实际的行列索引为(1, 3)，即第二行第四个元素，二维数据的总列数 = 5
// 返回的一位数组形式的索引为: 1*5 + 3 = 8
#define OFFSET(row, col, ld) ((row) * (ld) + (col))

// 定义naive gemm的kernel函数
__global__ void naiveSgemm(
    float * __restrict__ a, float * __restrict__ b, float * __restrict__ c,
    const int M, const int N, const int K) {
    
    // 当前thread在C矩阵中的row
    int m = blockIdx.y * blockDim.y + threadIdx.y;
    // 当前thread在C矩阵中的col
    int n = blockIdx.x * blockDim.x + threadIdx.x;
    if (m < M && n < N) {
        float psum = 0.0;
        // 告知编译器自动展开循环体，这样可以减少循环控制的开销（循环次数小的时候可以这么做）
        #pragma unroll
        // 取出A[row]和B[col]，然后逐个元素相乘累加，得到最终结果
        for (int k = 0; k < K; k++) {
            // a[OFFSET(m, k, K)]: 获取A[m][k]
            // b[OFFSET(k, n, N)]: 获取B[k][n]
            psum += a[OFFSET(m, k, K)] * b[OFFSET(k, n, N)];
        }
        c[OFFSET(m, n, N)] = psum;
    }
}

const int BM = 32, BN = 32;
const int M = 512, N = 512, K = 512;
dim3 blockDim(BN, BM);
dim3 gridDim((N + BN - 1) / BN, (M + BM - 1) / BM);
```

可想而知，由于这种办法要重复从global memory上读取数据，所以读取数据上消耗了大量时间，它肯定没有办法充足利用起GPU的算力。

## GEMM优化：矩阵分块，从global memory到SMEM

**on-chip内存的带宽要比off-chip内存的带宽大得多**。把矩阵A和B都搬运到on-chip的SMEM上，然后采用和naive GEMM一样的计算方法，那么尽管还是会在SMEM上发生重复读数据的情况（也即总的读写次数和naive一样，只不过现在不是从global memory读取，是从SMEM上读取），可是因为带宽变大了，总体来说数据读取时间肯定减少了。

**但是问题是，SMEM的存储要比global memory小很多，当矩阵比较大时，根本没办法把完整的矩阵搬运到SMEM上**。**如果搬运不了完整的矩阵，对矩阵切切块，搬运它的一部分**

![image-20240617201259058](GEMM.assets/image-20240617201259058.png)

- 把A矩阵横着切分成四块，每块大小为`(128, 512)`
- 把B矩阵纵着切分成四块，每块大小为`(512, 128)`

这里我们共有4*4 = 16个block，每个block负责计算C矩阵中大小为`128x128`的部分（图中绿色块）。

那么现在只需要把A的分块（红色）与B的分块（黄色）从global memory搬运到SMEM上，然后再从SMEM做一系列读取操作去计算C。**如此循环，直到所有的C分块都计算出来为止。**

**如果SMEM还是装不下红色和黄色块，那怎么办？**

继续切块。

![image-20240617204815603](GEMM.assets/image-20240617204815603.png)

上图中A矩阵的高亮红块`(128, 8)`，B矩阵中的高亮黄块`(8, 128)`，就是我们再切割的结果。

按照现在的划分，我们再来理一下**一个block**内做的事情：

- 每次取A矩阵的一个分块，取B矩阵的一个分块，将两者相乘得到分块矩阵C
- 对A矩阵，向右找到下一个分块；对B矩阵，向下找到下一个分块，然后再相乘得到分块矩阵C，累加到上一个分块矩阵C上。
- 如此循环，当我们遍历完所有的A分块和B分块后，就可以得到最终的分块矩阵C了。也就是我们图中的高亮绿块。

## GEMM再优化：从SMEM到Register

![image-20240617210846922](GEMM.assets/image-20240617210846922.png)

**为了更好利用SMEM，减少从global memory读数据，做了以下事情。**

**Global memory**

- 在global memory上，存放着用于计算的矩阵A，B；和结果矩阵C（初始化状态，还没被算出来）
- 我们不想从低带宽的global memory上一个个读数据，我们想多利用高带宽的SMEM。**因此我们设计了16个可以独立计算的block（绿色），每个block处理一块A（浅红色）与一块B（浅黄色）**。理想情况下，每个block计算时，它会将浅红和浅黄加载到SMEM上，然后做计算。
- 但是，浅红和浅黄，可能对于SMEM来说还是太大了。所以，我们选择再次切割，每个block做计算时，加载高亮红和高亮黄去SMEM上。

**SMEM**

- **单个block在做计算时，会有若干次循环**
- **在每次循环内，block会从global memory上加载一块高亮红和高亮黄到SMEM上**（每个thread加载这块高亮红和高亮黄的一部分），然后计算得到单次循环结果。所有循环结果累加，即得到这块block的最终结果（**split-by-k**）

以上两部分是对上文内容的总结，现在我们来看从SMEM -> Register的步骤

**Register**

- 单个block做单次循环时，实际负责计算的是它当中的threads，如上图，每个threads负责计算这个block内(Tm, Tn)大小的矩阵。
- 这个矩阵由上图右侧的浅红色块和浅黄色块加载而来，而这两个色块在SMEM上，也就是thread会从SMEM上逐一取数。
- 在on-chip的memory上，register是比SMEM带宽更高，存储更小的数据。
- 所以比起一次次从SMEM上读数，不如类比于global memory -> SMEM的思路，把数据切块后，加载到register中，再做计算。
- 所以，**单个block的单次循环下，单个thread也存在若干次循环。每次循环内，该thread从SMEM上读取渐变红和渐变黄色块到register，然后再做计算**，thread所有循环的结果相加，即得到该thread的最终结果（**split-by-k**）。

我们马上进入代码实践讲解，在此之前我们先比对上图，把矩阵的各个维度再明确下：

- M = N = K = 512
- Bm = Bn = 128
- Bk = 8
- Tm = Tn = 8

在单个block的单次循环内，计算某对高亮红和高亮黄时，block内线程的排布如下：

![image-20240617211457500](GEMM.assets/image-20240617211457500-1718798272309.png)



