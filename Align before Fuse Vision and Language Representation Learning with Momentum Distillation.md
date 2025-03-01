# [Align before Fuse: Vision and Language Representation Learning with Momentum Distillation](zotero://select/library/items/JYX2XMMM)

NeurIPS，2718次

<img src="https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227170757594.png" alt="image-20250227170757594"  />

## 模型架构

整体上，模型分为text encoder、image encoder、multimodal encoder三个部分。Momentum model是进行蒸馏的手段，即使去掉ALBEF也是完整的模型。

###  Encoder

图像encoder采用12层的ViT-B/16模型；文本encoder采用6层transformer，多模态encoder采用6层transformer，有趣的是这12层transformer就是BERT模型的12层transformer，前6层用作文本encoder，后6层用作多模态encoder。图像侧和文本侧都带有[cls] token。另外，多模态transformer采用cross attention的方式，融合图像token和文本token。

### Loss function

采用三种loss，然后加和。Image-text contrastive learning (ITC) on the unimodal encoders, masked language modeling (MLM) and image-text matching (ITM)。

#### ITC

为了在融合之前得到更好的单模态表示而做的对比学习，借鉴MoCo的思路，ALBEF维护两个队列，分别存储最近的M个图像特征和M个文本特征。该特征是[cls] embedding经过线性变换和Normalization以后生成的。ITC中使用的loss是标准的对比损失。

用softmax对原本的相似度进行平滑，之后用ground-truth y和p的cross-entropy计算，图片和文本加和后平均。

![image-20250227172247363](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227172247363.png)

![image-20250227172256613](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227172256613.png)

#### MLM

Bert中的思路，对文本mask然后用交叉熵计算损失。

![image-20250227172839641](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227172839641.png)

#### ITM

ITM是图像和文本匹配损失函数，用来预测图像和文本是否匹配。采用multimodal encoder的[cls] token来表示图文融合的特征，然后连接上FC和softmax，预测2类的概率值，表示图文对是否匹配。ALBEF还会使用ITC中的图文对比损失去挖掘hard negatives，从而增强ITM任务。

![image-20250227172810838](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227172810838.png)

### Momentum Distillation

动量模型由单模态和多模态编码器的指数移动平均版本组成。

图文对当中有噪声，正样本中，可能图片和文字相关性很弱。ITC中可能，负样本文字可能也与图片相符合，MLM中，替换的文字可能比原本的文字能更好地描述图片，但 one-hot ground truth忽视这样的相关性，惩罚所有的负样本。

利用**动量模型**（参数为历史模型的指数移动平均）生成更可靠的伪标签（软标签），指导当前模型训练，从而减少对噪声标签的依赖。动量模型对同一输入生成预测分布（软标签），作为监督信号。

训练时训练base model使其与动量模型中的预测匹配，ITC中，直接使用动量模型产生的相似度。

原始任务loss加上预测目标和伪目标散度的方式来重新定义原始任务损失。

![image-20250227173950258](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227173950258.png)

## 结果

![image-20250227174657416](https://houxiong-pictures.oss-cn-beijing.aliyuncs.com/image-20250227174657416.png)



