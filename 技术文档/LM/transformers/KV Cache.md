[KV Cache：LLM 推理加速 — Bookstall](https://bookstall.github.io/2023/12/17/kv-cache/#生成式-llm-的推理过程)

# 显存占用

在推理时新增了 `past_key_values` 参数，该参数就会以追加方式保存每一轮的 K、V 值。KV Cache 变量内容为 `((k,v), (k,v), ..., (k,v))`。

```python
# 比如 假定数据为float16类型
batch_size=32, layer=32, dim_size=4096, seq_length=2048, head=32, 
# 需要的显存为
2∗32∗32*4096∗2048∗2/1024/1024/1024/1024=32G
```

# 代码

目前各大深度学习框架都实现了 KV Cache。例如，HuggingFace 的 transformers 库中的 generate 函数已经将其封装，用户不需要手动传入 `past_key_values` 并默认开启（`config.json` 文件中 `use_cache=True`）。

下面以 HuggingFace Transformers 的 GPT-2 为例。

```python
query = self._split_heads(query, self.num_heads, self.head_dim) # Q
key = self._split_heads(key, self.num_heads, self.head_dim) # K
value = self._split_heads(value, self.num_heads, self.head_dim) # V

if layer_past is not None: # 当输出第一个 token 后，layer_past 就是非 None 了
    past_key, past_value = layer_past # 取出之前计算好的 key, value
    key = torch.cat((past_key, key), dim=-2) # past_key 与当前 token 对应的 key 拼接
    value = torch.cat((past_value, value), dim=-2) # past_value 与当前 token 对应的 value 拼接

if use_cache is True:
    present = (key, value)
else:
    present = None
```

GPT2原始的推理过程：

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer


model = GPT2LMHeadModel.from_pretrained("/WORK/Test/gpt", torchscript=True).eval()

# tokenizer
tokenizer = GPT2Tokenizer.from_pretrained("/WORK/Test/gpt")
in_text = "Lionel Messi is a"
in_tokens = torch.tensor(tokenizer.encode(in_text))	# 分词，然后得到每个词在词表中的Index

# inference
token_eos = torch.tensor([198]) # line break symbol
out_token = None
i = 0
with torch.no_grad():
    while out_token != token_eos:
        logits, _ = model(in_tokens) # logits是一组词（序列），其中包含每个词在词表中的概率 二维
        out_token = torch.argmax(logits[-1, :], dim=0, keepdim=True) # 取最后一个词 并选择最大概率的词下标 即得到一个index
        in_tokens = torch.cat((in_tokens, out_token), 0) # 将之前的词和新的到的词拼接作为下一次的输入
        text = tokenizer.decode(in_tokens)
        print(f'step {i} input: {text}', flush=True)
        i += 1

out_text = tokenizer.decode(in_tokens)
print(f' Input: {in_text}')
print(f'Output: {out_text}')
```

每次只得到一个词：

```shell
step 0 input: Lionel Messi is a player
step 1 input: Lionel Messi is a player who
step 2 input: Lionel Messi is a player who has
step 3 input: Lionel Messi is a player who has been
step 4 input: Lionel Messi is a player who has been a
step 5 input: Lionel Messi is a player who has been a key
step 6 input: Lionel Messi is a player who has been a key part
step 7 input: Lionel Messi is a player who has been a key part of
step 8 input: Lionel Messi is a player who has been a key part of the
step 9 input: Lionel Messi is a player who has been a key part of the team
step 10 input: Lionel Messi is a player who has been a key part of the team's
step 11 input: Lionel Messi is a player who has been a key part of the team's success
step 12 input: Lionel Messi is a player who has been a key part of the team's success.
step 13 input: Lionel Messi is a player who has been a key part of the team's success.

Input: Lionel Messi is a
Output: Lionel Messi is a player who has been a key part of the team's success.
```

加入KV Cache之后的推理过程：

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer


model = GPT2LMHeadModel.from_pretrained("/WORK/Test/gpt", torchscript=True).eval()

# tokenizer
tokenizer = GPT2Tokenizer.from_pretrained("/WORK/Test/gpt")
in_text = "Lionel Messi is a"
in_tokens = torch.tensor(tokenizer.encode(in_text))

# inference
token_eos = torch.tensor([198]) # line break symbol
out_token = None
kvcache = None
out_text = in_text
i = 0
with torch.no_grad():
    while out_token != token_eos:
        logits, kvcache = model(in_tokens, past_key_values=kvcache) # 增加了一个 past_key_values 的参数
        out_token = torch.argmax(logits[-1, :], dim=0, keepdim=True)
        in_tokens = out_token # 输出 token 直接作为下一轮的输入，不再拼接
        text = tokenizer.decode(in_tokens)
        print(f'step {i} input: {text}', flush=True)
        i += 1
        out_text += text

print(f' Input: {in_text}')
print(f'Output: {out_text}')
```

