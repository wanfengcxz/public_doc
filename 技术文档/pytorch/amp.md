
https://pytorch.org/tutorials/recipes/recipes/amp_recipe.html

## 简介


混合精度`torch.cuda.amp`允许一些op(eg. linear,conv)使用更快的float16或bfloat16精度来更快地执行，而类似reduction的op使用float32的精度执行。自动混合精度可以自动匹配op到其适合的精度，从而降低网络的运行时间和内存占用。

训练时主要使用`torch.autocast`和`torch.cuda.amp.GradScaler`。torch需要在1.6以上，以及nvidia-GPU。
在含有TensorCore的GPU(Volta, Turing, Ampere)上，加速可达到2-3X

## 模型
```python
def make_model(in_size, out_size, num_layers):
    layers = []
    for _ in range(num_layers - 1):
        layers.append(torch.nn.Linear(in_size, in_size))
        layers.append(torch.nn.ReLU())
    layers.append(torch.nn.Linear(in_size, out_size))
    return torch.nn.Sequential(*tuple(layers)).cuda()
```
batch_size, in_size, out_size, num_layers需要足够大，使GPU饱和，这样混合精度才能提供更大的加速。

改变size来看加速效率如何。
```python
batch_size = 512 # Try, for example, 128, 256, 513.
in_size = 4096
out_size = 4096
num_layers = 3
num_batches = 50
epochs = 3

device = 'cuda' if torch.cuda.is_available() else 'cpu'
torch.set_default_device(device)

# Creates data in default precision.
# The same data is used for both default and mixed precision trials below.
# You don't need to manually change inputs' ``dtype`` when enabling mixed precision.
data = [torch.randn(batch_size, in_size) for _ in range(num_batches)]
targets = [torch.randn(batch_size, out_size) for _ in range(num_batches)]

loss_fn = torch.nn.MSELoss().cuda()
```