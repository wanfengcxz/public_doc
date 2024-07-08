# 概述

大模型第一定律

![image-20240625170230185](transformers.assets/image-20240625170230185.png)

- transformer的encoder结构取出来，即BERT系列模型
- decoder结构取出来，即GPT系列模型。专注于做生成任务

## LLM创新

![image-20240625182657227](transformers.assets/image-20240625182657227.png)

# LLaMA

[第十五课：LLaMA_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nN41157a9/?spm_id_from=333.337.top_right_bar_window_history.content.click&vd_source=b9f75d9bc23add3c1afdb0c760a8431b)

 LLaMA-13B > GPT-3-175B

LLaMA的优点

- 单GPU可运行
- 开源公开数据集训练
- 开源

![image-20240625180716107](transformers.assets/image-20240625180716107.png)

LLaMA的家族树

![image-20240625181149231](transformers.assets/image-20240625181149231.png)

## PositionEncoding

RoPE旋转位置编码 相对和绝对信息的融合 用绝对位置编码表征相对位置编码

LLM如何处理超长序列？一般训练时会固定上下文长度，例如1024个token。但有时需要模型生成很长的数据，例如一篇小说等。这涉及到模型的外推能力。

![image-20240625185014764](transformers.assets/image-20240625185014764.png)

![image-20240625190330213](transformers.assets/image-20240625190330213.png)

第m个token和第n个token做点积时，结果只和(m-n)有关

![image-20240625190633524](transformers.assets/image-20240625190633524.png)

![image-20240625201237243](transformers.assets/image-20240625201237243.png)

![image-20240625201423256](transformers.assets/image-20240625201423256.png)

![image-20240625201628465](transformers.assets/image-20240625201628465.png)

## Attention



## Norm

- Norm的位置发生变化

![image-20240625183403861](transformers.assets/image-20240625183403861.png)

- 采用RMSNorm

![image-20240625183846863](transformers.assets/image-20240625183846863.png)

![image-20240625184355589](transformers.assets/image-20240625184355589.png)

## FFN

激活函数改为SwiGLU

![image-20240625184618926](transformers.assets/image-20240625184618926.png)