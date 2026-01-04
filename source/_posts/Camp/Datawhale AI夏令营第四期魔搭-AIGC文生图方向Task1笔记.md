---
title: Summer_Camp
date: 2024-8-8 12:00:00
toc_number: true
tags: 
  - Camp
categories: 
  - Memo
description: 用魔搭将一个故事吧！
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648667.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736669.jpg
abbrlink: 7
---

## Datawhale AI夏令营第四期魔搭-AIGC文生图方向Task1笔记

#### 文生图技术时间轴：https://www.bilibili.com/video/BV12s4y1o7ys?vd_source=0dd681bfd61a5b268c459718c85af23a

| **发展阶段**                              | **发展介绍**                                                 |
| ----------------------------------------- | ------------------------------------------------------------ |
| **早期探索**（20世纪60年代-20世纪90年代） | 文生图的概念最早出现于计算机视觉和图像处理的早期研究中。早期的图像生成技术主要依赖于规则和模板匹配，通过预定义的规则将文本转换为简单的图形。然而，由于计算能力和算法的限制，这一阶段的技术能力非常有限，生成的图像质量较低，应用场景也非常有限。 |
| **基于统计模型的方法**（2000年代）        | 进入2000年代，随着统计模型和机器学习技术的发展，文生图技术开始得到更多关注。研究者们开始利用概率图模型和统计语言模型来生成图像。尽管这一阶段的技术在生成图像的多样性和质量上有了一定提升，但由于模型的复杂性和计算资源的限制，生成的图像仍然较为粗糙，不够逼真。 |
| **深度学习的崛起**（2010年代）            | 2010年代是文生图技术发展的一个重要转折点。随着深度学习，尤其是**卷积神经网络**（CNN）和**生成对抗网络**（GAN）的发展，文生图技术取得了突破性进展。2014年，Goodfellow等人提出的GAN模型通过生成器和判别器的对抗训练，极大地提升了图像生成的质量。随后，各类变种GAN模型被提出，如DCGAN、Pix2Pix等，使得文生图技术在生成逼真图像方面达到了前所未有的高度。![img](https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081459007.png) |
| **大规模预训练模型**（2020年代）          | 进入2020年代，大规模预训练模型如`OpenAI`的CLIP、DALL-E以及Stable Diffusion等的出现，标志着文生图技术进入了一个新的时代。CLIP通过大规模的文本和图像配对数据训练，能够理解和生成高度一致的文本和图像；DALL-E和Stable Diffusion进一步提升了生成图像的创意和细节表现能力，使得通过简单的文本描述生成高质量、复杂图像成为可能。这些技术的应用范围从艺术创作、广告设计到辅助医疗诊断，展现了广泛的商业价值和社会影响力。 |



### 概念解析：

* **扩散模型：**扩散模型是一类生成模型，它运用了物理热力学中的扩散思想，主要包括前向扩散和反向扩散两个过程。

  **详细解释** https://zhuanlan.zhihu.com/p/651103343

* **卷积神经网络：**卷积神经网络（Convolutional Neural Network）是一类包含卷积计算且具有深度结构的前馈神经网络，专门用来处理具有类似网格结构的数据的神经网络。

  **图解讲述** https://www.bilibili.com/video/BV1x44y1P7s2?vd_source=0dd681bfd61a5b268c459718c85af23a

  **详细解释** [卷积神经网络（convolutional neural network, CNN） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/616349739?utm_id=0)
  
* **生成对抗网络：**GAN（Generative Adversarial Network）通过两个神经网络 —— 生成器（Generator）和判别器（Discriminator）的相互对抗来进行学习，从而生成逼真的数据。

  **详细解释 **https://blog.csdn.net/m0_70066267/article/details/140661019

  **生动讲述** https://www.bilibili.com/video/BV1nA4m1N74j?vd_source=0dd681bfd61a5b268c459718c85af23a

### 代码详解

#### 环境安装：引入必要库（install后除了'-' + 字母外（表示安装选项）的名称均为库）

```python
!pip install simple-aesthetics-predictor
# simple-aesthetics-predictor：其是一个用于预测图像美学质量的Python库
# 通常使用深度学习模型来分析图像，并给出一个美学评分，反映图像的视觉吸引力

!pip install -v -e data-juicer
# Data-Juicer：数据处理和转换工具，旨在简化数据的提取、转换和加载过程

!pip uninstall pytorch-lightning -y

!pip install peft lightning pandas torchvision
# 更新lightning —— 以前称为pytorch-lightning
# lightning：其是一个用于加速和简化PyTorch代码的深度学习框架
# 它提供了一种结构化的方式来组织PyTorch代码，使得代码更加模块化和易于维护
# pandas：用于数据操作和分析
# torchvision：提供与PyTorch深度学习框架一起使用的计算机视觉工具

!pip install -e DiffSynth-Studio
# DiffSynth-Studio：高效微调训练大模型工具
```

**效果预览**

![Snipaste_2024-08-08_14-22-44](https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081454305.png)

#### 下载数据集

```python
from modelscope.msdatasets import MsDataset
# modelscope 是一个开源的人工智能平台，提供了丰富的预训练模型、数据集、工具和示例

ds = MsDataset.load(
    'AI-ModelScope/lowres_anime',
    subset_name='default', 
    split='train', 
    cache_dir="/mnt/workspace/kolors/data"
)
# 从modelscope平台加载名称为'AI-ModelScope/lowres_anime'的数据集的子集'default'的'train'部分
```

> 无输出

#### 加载数据集

```python
import json, os
from data_juicer.utils.mm_utils import SpecialTokens
from tqdm import tqdm
# json：用于处理JSON数据
# os：用于文件和目录操作
# tqdm：用于显示循环进度条

os.makedirs("./data/lora_dataset/train", exist_ok=True) # ./表示当前目录
os.makedirs("./data/data-juicer/input", exist_ok=True)
# 使用os.makedirs创建两个目录结构：
# exist_ok=True 参数表示如果目录已存在，不会抛出错误。
with open("./data/data-juicer/input/metadata.jsonl", "w") as f:
    for data_id, data in enumerate(tqdm(ds)):
        # 打开文件'./data/data-juicer/input/metadata.jsonl'
        image = data["image"].convert("RGB")
        # 将图像转换为RGB格式
        image.save(f"/mnt/workspace/kolors/data/lora_dataset/train/{data_id}.jpg")
        # 将图像保存到指定目录，并为每个数据分配一个唯一的data_id
        metadata = {"text": "二次元", "image": [f"/mnt/workspace/kolors/data/lora_dataset/train/{data_id}.jpg"]}
        # 创建元数据字典，字典可以理解为双射，表示一一对应的关系
        f.write(json.dumps(metadata))
        # 将上述字典JSON字符串并写入文件（json是一种数据集常用的文本格式）
        f.write("\n")
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081455381.png" alt="image-20240808142347571" style="zoom:67%;" />

#### 处理数据集，保存数据处理结果

```python
data_juicer_config = """
# global parameters 全局变量
project_name: 'data-process'
dataset_path: './data/data-juicer/input/metadata.jsonl'  # 数据集路径
np: 4  # 子进程数量，用于并行处理数据集

text_keys: 'text' # 文本字段的键，设置为text
image_key: 'image' # 图像字段的键，设置为image
# 上一个代码段中写入字典（metadata）中设置键 - 值
image_special_token: '<__dj__image>' # 图像特殊标记

export_path: './data/data-juicer/output/result.jsonl' # 导出的结果文件路径

# 处理计划
process: 
    - image_shape_filter: # 处理图像尺寸
        min_width: 1024
        min_height: 1024
        any_or_all: any
    - image_aspect_ratio_filter: # 处理图像长宽比
        min_ratio: 0.5
        max_ratio: 2.0
        any_or_all: any
"""
with open("data/data-juicer/data_juicer_config.yaml", "w") as file:
    file.write(data_juicer_config.strip())
# 将配置内容写入一个YAML文件
!dj-process --config data/data-juicer/data_juicer_config.yaml
# 运行dj-process命令处理数据集，并指定使用生成的配置文件
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081455641.png" alt="image-20240808142457518" style="zoom:50%;" />

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081455927.png" alt="image-20240808142516824" style="zoom:50%;" />

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081456637.png" alt="image-20240808142559471" style="zoom:50%;" />

#### 加载数据集

```python
import pandas as pd
import os, json
from PIL import Image
from tqdm import tqdm
# PIL.Image: 用于图像处理。

texts, file_names = [], []
os.makedirs("./data/lora_dataset_processed/train", exist_ok=True)
with open("./data/data-juicer/output/result.jsonl", "r") as file:
    for data_id, data in enumerate(tqdm(file.readlines())):
        data = json.loads(data)
        text = data["text"]
        texts.append(text)
        image = Image.open(data["image"][0])
        image_path = f"./data/lora_dataset_processed/train/{data_id}.jpg"
        image.save(image_path)
        file_names.append(f"{data_id}.jpg")
# 将上一个代码段处理的结果（储存在'result.jsonl'内）重新存储在'image_path'当中
# 保存图片文本字段的键和存储的图片地址在'texts'和’file_names'中
data_frame = pd.DataFrame()
data_frame["file_name"] = file_names
data_frame["text"] = texts
data_frame.to_csv("./data/lora_dataset_processed/train/metadata.csv", index=False, encoding="utf-8-sig")
data_frame
# 将 file_names 和 texts 列表添加到 DataFrame 中，分别作为 file_name 和 text 列
# DataFrame可以理解为表格
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081456819.png" alt="image-20240808142647059" style="zoom:67%;" />

#### 下载模型

```python
from diffsynth import download_models

download_models(["Kolors", "SDXL-vae-fp16-fix"])
# 下载模型至'kolors'目录下的'models'文件夹下
```

**效果预览**															

​																												.													

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081456753.png" alt="image-20240808142728416" style="zoom:67%;" />

#### 查看训练脚本的输入参数

```python
!python DiffSynth-Studio/examples/train/kolors/train_kolors_lora.py -h
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081456605.png" alt="image-20240808142808448" style="zoom:67%;" />

#### lora微调

> **概念解析：**
>
> `LoRA`（Low-Rank Adaptation）是一种高效的模型微调方法，旨在减少在大规模预训练模型上进行微调时的计算和存储开销。LoRA 的核心思想是通过在模型的特定层中引入低秩矩阵，来减少模型参数的调整范围，从而实现更高效的微调。

```python
import os
# os - 操作系统，提高与操作系统交互的能力，比如运行如下的运行系统程序

cmd = """
python DiffSynth-Studio/examples/train/kolors/train_kolors_lora.py \ # 指定要运行的 Python 脚本
  --pretrained_unet_path models/kolors/Kolors/unet/diffusion_pytorch_model.safetensors \
  --pretrained_text_encoder_path models/kolors/Kolors/text_encoder \
  --pretrained_fp16_vae_path models/sdxl-vae-fp16-fix/diffusion_pytorch_model.safetensors \
  # 指定预训练模型的路径
  --lora_rank 16 \
  # 设置 LoRA 的秩为 16
  --lora_alpha 4.0 \
  # 设置 LoRA 的 alpha 参数为 4.0
  --dataset_path data/lora_dataset_processed \
  # 指定数据集的路径
  --output_path ./models \
  # 指定训练模型的输出路径
  --max_epochs 1 \
  # 设置最大训练轮数为 1
  --center_crop \
  # 启用中心裁剪
  --use_gradient_checkpointing \
  # 启用梯度检查点
  --precision "16-mixed"
  # 设置训练的数值精度为混合 16 位精度
""".strip()
# 讲句实话，我不是很懂上述命令行的运行逻辑，但是可以通过命令名字大致理解一下

os.system(cmd)
# 执行上述命令行
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081456146.png" alt="image-20240808143002792" style="zoom:67%;" />

#### 加载微调好的模型

```python
from diffsynth import ModelManager, SDXLImagePipeline
from peft import LoraConfig, inject_adapter_in_model
import torch
# 从'diffsynth'引入'ModelManager', 'SDXLImagePipeline'用于模型管理和图像生成任务
# 从'peft'中引入用于'LoraConfig', 'inject_adapter_in_model'配置和注入'LoRA'适配器到深度学习模型中。
# 'torch'主要用于深度学习的相关操作，此处用于模型的加载和参数管理

def load_lora(model, lora_rank, lora_alpha, lora_path):
    lora_config = LoraConfig(
        r=lora_rank,
        lora_alpha=lora_alpha,
        init_lora_weights="gaussian",
        target_modules=["to_q", "to_k", "to_v", "to_out"],
    )
    # 配置 LoRA 适配器的参数：秩，权重调节参数，初始化权重，模块（'to_q'—查询矩阵）、'to_k' — 键矩阵、'to_v' — 值矩阵、 'to_out' —输出矩阵
    model = inject_adapter_in_model(lora_config, model)
    # 将上述设置的 LoRA 适配器注入到预训练的模型中（model为调用函数时传入的参数）
    state_dict = torch.load(lora_path, map_location="cpu")
    # 从'lora_path'中加载已设置好的模型权重和其他数据（此处是加载到CPU上）
    model.load_state_dict(state_dict, strict=False)
    # 将已加载的权重（即 state_dict）应用到模型上
    return model


# Load models
model_manager = ModelManager(torch_dtype=torch.float16, device="cuda",
                             file_path_list=[
                                 "models/kolors/Kolors/text_encoder",
                                 "models/kolors/Kolors/unet/diffusion_pytorch_model.safetensors",
                                 "models/kolors/Kolors/vae/diffusion_pytorch_model.safetensors"
                             ])
# 将前面已下载的模型加载到程序中，用于接下来的任务
# 配置模型的精度（如 16 位浮点数），16精度可保证参数不是很复杂，便于训练（32精度就复杂啦）
# 配置计算设备（GPU）,我们在阿里云平台上租用的是GPU
pipe = SDXLImagePipeline.from_model_manager(model_manager)
# pipe 集成了 ModelManager 加载的多个模型（文本编码器、UNet、VAE），以便执行图像生成任务
# 文本编码器：将文本提示转换为向量表示
# UNet：使用这些向量表示生成图像
# VAE：捕捉和生成图像数据的潜在表示，以增强生成模型的能力

# Load LoRA
pipe.unet = load_lora(
    pipe.unet,
    lora_rank=16, # This parameter should be consistent with that in your training script.
    lora_alpha=2.0, # lora_alpha can control the weight of LoRA.
    lora_path="models/lightning_logs/version_0/checkpoints/epoch=0-step=500.ckpt" # 预训练模型的权重
)
# 得到最终的模型！ pipe.unet
```

**效果预览**

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081457289.png" alt="image-20240808143151435" style="zoom:67%;" />

#### 图片生成

```python
torch.manual_seed(0)
# 使用torch生成随机种子
# 这里补充一下概念：随机种子，使用随机种子可保证随机性与结果的可重复性。、
# 在深度学习中，随机性会影响模型的初始化和生成过程，通过设置种子，可以保证每次运行代码生成的结果是一致的。

image = pipe(
    prompt="二次元，一个紫色短发小女孩，在家中沙发上坐着，双手托着腮，很无聊，全身，粉色连衣裙",
    # 正面提示词：你想要图片是怎么怎样的，建议为意义明确的短语
    # 建议仿照上述格式，从整体到局部（提示词的效果是从前往后递减的）
    negative_prompt="丑陋、变形、嘈杂、模糊、低对比度",
    # 负面提示词：你不想要图片是怎样的
    cfg_scale=4, 
    # 分类器自由引导尺度：通俗来说，数值越高，提示词对模型的约束更强
    num_inference_steps=50, 
    # 推理步骤的数量。更多的步骤通常会生成质量更高的图像，但也会增加计算时间。
    height=1024, width=1024, # 生成图片的尺寸
)
image.save("1.jpg")
# 最后生成图像！
```

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408081457153.png" alt="image-20240808143216197" style="zoom:67%;" />

### 成片展示：小猫咪和青年的同居生活

* 正向提示词：卡通，一个可爱的灰色小猫咪，在一个圆形睡垫睡着，头埋在胸前，很悠闲，全身，处在一个房间内，毛发柔顺
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121532349.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个毛发柔顺的灰色小猫咪躺在圆形睡垫上，抬头望向门口，露出惊喜的神情
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121532350.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个穿着休闲白衬衫和牛仔裤的青年打开门，低头看向屋内，一只毛发柔顺的灰色小猫咪，睡在睡垫
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度，色情擦边

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121532351.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个毛发柔顺的灰色小猫咪，从屋内跑向门口,开心
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121611489.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个穿着休闲白衬衫和牛仔裤的青年蹲在地上，给一个毛发柔顺的灰色小猫咪喂食，小猫咪贴近青年的手吃食物
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121532352.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个穿着休闲白衬衫和牛仔裤的青年坐在沙发上，青年用手抚摸怀中的一个毛发柔顺的灰色小猫咪，面前有一台电视
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121532348.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个穿着休闲白衬衫和牛仔裤的青年给一个灰色小猫咪洗澡，青年帅气，白净，猫咪可爱，毛发柔顺
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121623640.jpg" alt="image-20240808143216197" style="zoom:67%;" />

* 正向提示词：卡通，一个穿着休闲白衬衫和牛仔裤的青年抱着一个灰色小猫咪睡在床上，年帅气，白净，猫咪可爱，毛发柔顺
* 负面提示词：丑陋、变形、嘈杂、模糊、低对比度

<img src="https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121623641.jpg" alt="image-20240808143216197" style="zoom:67%;" />

