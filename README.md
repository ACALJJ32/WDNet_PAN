# 百度网盘AI大赛——图像处理挑战赛：文档图像摩尔纹消除第3名方案
## 简介
> 这是我们参加百度去摩尔纹比赛[competition](https://aistudio.baidu.com/aistudio/competition/detail/128/0/introduction)的代码与模型，主要参考了[WDNet](https://arxiv.org/abs/2007.07173)以及[PAN](https://arxiv.org/pdf/2010.01073.pdf)，在官方提供的[baseline](https://aistudio.baidu.com/aistudio/projectdetail/3220041)的基础上进行改进与创新.

## 网络结构
> ![](https://ai-studio-static-online.cdn.bcebos.com/14f53ba3ad9249608da1a2f642cc9ada481d3fc7d1ea449ca17ffb268594889d)
我们首先是将RGB图片通过WaveletTransform模块进行转换，得到一个48通道的数据，通过改进的WDNet网络同样得到一个通道数与尺寸不变的特征图。最后在一次通过WaveletTransform使用转置卷积将图片还原得到最终预测结果。WaveletTransform的权重是固定不变的。  
在改进的WDNet训练好之后，用训练好的模型对训练数据做一次推理预测，用来生成新的训练数据，供第二阶段的PAN模型进行训练。 


### 改进的WDNet
> 考虑到摩尔纹图案的尺度/频率跨度较大，所以想到用金字塔结构来解决多尺度问题。   ![](https://ai-studio-static-online.cdn.bcebos.com/a20d31f52b6a4e759ac1a48551f15bd2d42108852bed491a87ed0aa2fc027e72)
我们首先使用两个残差模块对48通道的数据进行特征提取，然后用Downsample conv进行下采样，分三个分支进行特征传播；我们同时还引入了Spatial Attention Branch来代替之前的Dilation Branch，用来更好地提取保留原始特征的信息。

### Dense Branch
> 网络结构：
![](https://ai-studio-static-online.cdn.bcebos.com/9124c6d18b3b462c84f61d777462c547261c2397a6ea4ec8ab491909ff05ac18)
我们依然保留了Baseline的Dense Block模块，因为在改进的过程中发现该模块较难改动，是影响性能的核心模块，经过多次实验后，我们决定对该模块进行保留。在不改动Dense Block的基础上，我们用Spatial Attention Branch代替原来的Dilation Branch，增强图像中的细节。

#### Spatial Attention Branch
> 网络结构：
![](https://ai-studio-static-online.cdn.bcebos.com/c5a6d6dac88b4faf91c47df4537a4048863ba08e773e44babf4c16c1e54c06e1)
这部分的主要是通过两种pooling：max pooling和avg pooling提取特征的空间信息，更好地保存原始特征的空间信息。

### PAN  
> 网络结构：
![](https://ai-studio-static-online.cdn.bcebos.com/b68a5df0dd734c4dafcd6c10fefb2bb891d0539eea694300a1483e89c5555778)
这部分网络主要参考了图像超分辨率模型[PAN](https://arxiv.org/pdf/2010.01073.pdf)，我们在一阶段的实验中发现，实验结果中的字体细节有不同程度的丢失，所以我们想找一种轻量的超分辨率网络模型，进一步优化一阶段的实验结果，同时做到去模糊与去摩尔纹。

## 代码组织结构介绍  
> work   
&emsp;|-----model.py  
&emsp;|-----net_utils.py  
&emsp;|-----predict.py    
&emsp;|-----train.py  
&emsp;|-----train_stage2.py  
&emsp;|-----wdnet_predict.py  
&emsp;|-----  ......  
&emsp;|-----output  
&emsp;&emsp;&emsp;&emsp;&emsp;|-----pre  
&emsp;|-----weight  
&emsp;&emsp;&emsp;&emsp;&emsp;|-----pwdnet_epoch_395   
&emsp;&emsp;&emsp;&emsp;&emsp;|-----epoch_980  
train.py用来运行训练改进的WDNet；train_stage2.py用来训练PAN模型；model.py中存放的是WDNet模型，net_utils.py中放一些网络其他层的实现；wdnet_predict.py用来进行第一阶段的预测；predict.py用来加载改进的WDNet和PAN，一起进行推理；weight中存放我们已经训练好的模型；output/pre下面存放推理预测的结果。


## 数据增强/清洗策略  
> 我们并没有使用数据清洗；在数据增强方面，我们主要是将原baseline的resize改成了随机裁剪，保留了随机水平翻转。
![](https://ai-studio-static-online.cdn.bcebos.com/dd5bd1ed951645fc9bc4b28c3b0f98caae53f29a6bc546a988cd1ac5a29cc239)  


## 调参优化策略  
### WDNet的训练策略
> 首先，输入图像的patch=128，batch=16，init lr=2e-4，使用Adam优化，训练400个epoch；在收敛后进行调整：patch=256，batch=8，init lr=2e-4，用load_pretrained_model加载上一次训练最好的模型，继续训练；收敛后进行最后一次调整：patch=512，batch=4，init lr=1e-4，训练到收敛。  
### PAN的训练策略  
> 首先，输入图像的patch=128，batch=16，init lr=1e-4，使用Adam优化，训练2000个epoch；在收敛后进行调整：patch=256，batch=8，init lr=1e-4，用load_pretrained_model加载上一次训练最好的模型，继续训练；收敛后进行最后一次调整：patch=512，batch=4，init lr=1e-4，训练到收敛。  


## 安装运行环境
  ```
  pip install -r requirements.txt
  ```

## 下载预训练模型
> Baidu disk: https://pan.baidu.com/s/1Q1sEWf-9cVM83gj6wcXowg  
password: yt86

## 训练
> 
  ``` python
  python train.py  # train wdnet
  python train_stage2.py # train pan
  ```

## 推理预测
  ```python
  python predict.py
  ```

## AI Studio链接
> https://aistudio.baidu.com/aistudio/projectdetail/3439026?shared=1
