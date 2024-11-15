E2E Throughput (End-to-End Throughput)
定义：指整个推理过程的吞吐量，包括输入预处理、模型推理、输出后处理等所有步骤。
范围：覆盖整个系统的推理流程。
评价对象：用于评估系统的整体性能。
单位：通常以每秒处理的输入数量（如样本/秒）来衡量。
Decoder Throughput
定义：指解码器部分的吞吐量，仅衡量模型中解码器的处理速度。
范围：通常只涵盖模型核心的推理部分，不包括数据准备和结果处理。
评价对象：用于评估模型内部特定组件（如解码器）的效率。
单位：同样以每秒处理的输入数量（如样本/秒）来衡量。





在大模型推理中，**contiguous batching** 是一种优化技术，用于提高计算效率和资源利用率。

### 主要思想

- **批处理（Batching）**：将多个输入合并成一个批次进行处理，可以充分利用硬件的并行计算能力。
- **连续性（Contiguous）**：确保批次中的数据在内存中是连续存储的，减少内存访问时间，提高缓存命中率。

### 优势

1. **提高吞吐量**：通过批量处理多个输入，减少每个输入单独处理的开销。
2. **优化内存访问**：连续的内存布局有助于提升访问速度。
3. **提高硬件利用率**：更好地利用 GPU 的并行计算能力。

### 应用场景

- 深度学习模型的推理阶段，特别是需要处理大量小输入时。
- 实时系统中，通过批处理提高响应速度。

这种技术在需要高效推理的大型模型中尤为重要