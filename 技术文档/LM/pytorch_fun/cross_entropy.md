https://zhuanlan.zhihu.com/p/497672789

https://fengchao.pro/blog/pytorch-cross-entropy-loss/

使用pytorch逐步实现CrossEntropyLoss

```python
import torch

# 创建 logits（模型输出的原始分数）
logits = torch.tensor([[0.3, 0.1], [-0.4, 0.7], [-0.1, 0.8]])

print("原始 logits:")
print(logits)

# 创建标签
labels = torch.tensor([0, 0, 1])

print("labels:")
print(labels)

"""
原始logits:
tensor([[ 0.3000,  0.1000],
        [-0.4000,  0.7000],
        [-0.1000,  0.8000]])

labels:
tensor([0, 0, 1])
"""


# 应用 Softmax 函数
exp_logits = torch.exp(logits)
sum_exp_logits = torch.sum(exp_logits, axis=1, keepdim=True)
probabilities = exp_logits / sum_exp_logits

# 打印 Softmax 概率
print("\nSoftmax 概率：")
print(probabilities)

"""
Softmax概率:
tensor([[0.5498, 0.4502],
        [0.2497, 0.7503],
        [0.2891, 0.7109]])
"""

# 初始化损失值累加器
loss_accumulator = 0.0

# 用 for 循环计算每个样本的损失
for i in range(len(labels)):
    # 1/2步骤可以交换
    # 获取第 i 个样本的真实类别的概率
    prob = probabilities[i, labels[i]]	# 1
    # 计算第 i 个样本的负对数似然损失
    loss = -torch.log(prob)	# 2
    # 打印第 i 个样本的损失
    print(f"\n样本{i}的损失值：")
    print(loss)
    # 累加损失
    loss_accumulator += loss

# 计算平均损失
loss = loss_accumulator / len(labels)

# 打印平均损失值
print("\n平均损失值：")
print(loss)

"""
样本0的损失值:
tensor(0.5981)

样本1的损失值:
tensor(1.3873)

样本2的损失值:
tensor(0.3412)

平均损失值:
tensor(0.7755)
"""
```

pytorch的API

```python
# 使用 PyTorch 的 CrossEntropyLoss
criterion = torch.nn.CrossEntropyLoss()
torch_loss = criterion(logits, labels)

print(f"PyTorch 计算的损失值：{torch_loss}")

"""
PyTorch计算的损失值: 0.775542676448822
"""
```

