# Mask R-CNN

Faster R-CNN 的扩展，在其基础上增加了生成 segmentation mask 的功能。除了生成 mask，它还能扩展到其他的很多功能，比如估计人姿态等。

### 1. Mask R-CNN

- 第一阶段和 Faster R-CNN 一样，是 RPN。
- 第二阶段相比 Faster R-CNN 增加了一个**输出二值 mask 的分支**（和分类、bbox 回归分支并行）
- 训练阶段使用多任务 loss 函数：$L = L_{cls} + L_{box} + L_{mask}$ 。$L_{cls}$ 和 $L_{box}$ 的表达式和 Fast R-CNN 一样。$L_{mask}$ 针对每一类定义，对每个像素使用 sigmoid 并使用二值的交叉熵损失函数（相当于对每个像素做 logistic 回归）。
- $L_{mask}$ 的定义方法避免了不同类别之间的竞争。（传统方法—FCN—使用的是针对多类的多项式交叉熵损失）
- $L_{mask}$ 的计算使用了分类分支的输出类别，按照这个分类类别选择输出 mask。这种方法使得 mask 预测和分类过程解耦。

### 2. RoIAlign

- 在 Faster R-CNN 里面使用的提取 RoI 方法是 RoIPooling。RoIPooling 的步骤中存在量化误差：RoIPooling 首先把 bounding box 映射到特征图上的坐标**取整**（见 SPP net 中的坐标映射方法）；然后一个 RoI 对应的特征图被分成更小的窗口（Fast R-CNN 中分为 $7\times 7$），在这个过程中窗口的大小也被**取整**。由于量化误差的存在，导致特征图上实际映射的区域和真实的 RoI 不完全对应。对于普通的分类任务，小的平移可以被忽略；但是对于输出像素精度 mask 的任务来说，这样的 RoI 偏移将对结果有极大的影响。
- 为了解决这个问题，Mask R-CNN 提出了用 RoIAlign 层来代替 RoIPooling 层。两者的差别非常简单：RoIAlign 中去掉了所有的取整，无论是 bounding box 映射到特征图上的坐标还是对窗口大小的分割都直接计算精确的浮点数结果。然后，对于所有由浮点数定义边界的小窗口，从边界开始每隔 1 进行一次采样，对每个采样位置用双插值算法计算出该采样位置在特征图上的精确取值。对于每个小窗口上的所有采样值进行融合（max 或者 avg），得到每个小窗口的输出结果。最后的输出结果长度和 RoIPooling 的输出长度一样。
- RoIAlign 方法不改变原窗口的位置和形状。这种方法比 RoIPooling 效果好很多。（下文有结果比较）

### 3. mask 

- 用全卷积网络从每个 RoI 中预测 $m\times m$ 的 mask 。用全卷积层的原因是需要保留 mask 的空间位置信息。如果用全连接层的话会把特征提取为低维度的向量，缺少位置信息。

### 4. 网络结构

- 下面的网络结构用如下的描述方法定义：
  - Faster R-CNN 和 RPN 共享权值的卷积网络：backbone（网络-层数-特征图所在层）
  - 用来做分类和 bbox 回归的网络：head
  - 应用在每个 RoI 上，用来做 mask prediction 的网络。

- 论文中使用的 backbone 有这样几种：ResNet-50-C4 (第 4 段的卷积层输出) / ResNet-101-C4 / ResNet-50-FPN (Feature Pyramid Network) / ResNet-101-FPN / ResNeXt-101-FPN。使用 FPN 的网络可以提升准确率和速度。

- 论文中使用的 head 在两种前人设计的网络结构的基础上增加了 mask 分支：

  - ResNet：RoI --> 7 \* 7 \* 1024 --(res5)--> 7 \* 7 \* 2048 --(avg pooling)--> 2048 --(fc)-->cls, box

    ​                                                                                            |

    ​                                                                                             -----> 14 \* 14 \* 256 --> 14 * 14 * 80 --> mask

  - FPN：RoI --> 7 * 7 * 256 --(fc)--> 1024 --(fc)--> 1024 --(fc)--> 1024 --(fc)--> cls, box

    ​	   RoI --> 14 * 14 * 256 --(x4)--> 14 * 14 * 256 --> 28 * 28 * 256 --> 28 * 28 * 80 -->mask

  - 箭头代表卷积层（经过之后维度不变）或解卷积层（经过之后维度增加），res5 代表 ResNet 的第 5 段，x4 代表4个连续的卷积层。

### 5. 实现细节

- 训练过程中正样本的选择为与 ground truth box 的 IoU 大于 0.5 的候选框
- mask 损失函数只作用于正样本；mask 的训练 label 是候选框和 ground truth 的交集
- 训练样本的采样方法和 Fast R-CNN 一样，先采样图片后采样候选框，正负样本框比例为1:3
- RPN anchors 共有 15 种，5 种 scale，3 种长宽比
- 为了方便实现，RPN 和 Mask R-CNN 不共享卷积网络参数
- 测试过程中 C4 backbone 网络选取 300 个候选框，FPN 选取 1000 个候选框；box 回归分支作用于这些候选框，之后进行 NMS 。mask 分支作用于得分最高的 100 个检测框。mask 分支在每个候选框上输出 K 个 mask，但是只取第 k 个，k 为分类分支输出的类别编号。mask 浮点输出 resize 为 RoI 大小，以0.5 为

### 6. 实验结果

- instance segmentation

|   Method   |    backbone     |    AP    | AP$_{50}$ | AP$_{75}$ | AP$_{S}$ | AP$_{M}$ | AP$_{L}$ |
| :--------: | :-------------: | :------: | :-------: | :-------: | :------: | :------: | :------: |
|  FCIS+++   |  ResNet-101-C5  |   33.6   |   54.5    |     -     |    -     |    -     |    -     |
| Mask R-CNN |  ResNet-101-C4  |   33.1   |   54.9    |   34.8    |   12.1   |   35.6   |   51.1   |
| Mask R-CNN | ResNet-101-FPN  |   35.7   |   58.0    |   37.8    |   15.5   |   38.1   |   52.4   |
| Mask R-CNN | ResNeXt-101-FPN | **37.1** | **60.0**  | **39.4**  | **16.9** | **39.9** | **53.5** |

- 架构比较

|    backbone     |    AP    | AP$_{50}$ | AP$_{75}$ |
| :-------------: | :------: | :-------: | :-------: |
|  ResNet-50-C4   |   30.3   |   51.2    |   31.5    |
|  ResNet-101-C4  |   32.7   |   54.2    |   34.3    |
|  ResNet-50-FPN  |   33.6   |   55.2    |   35.3    |
| ResNet-101-FPN  |   35.4   |   57.3    |   37.5    |
| ResNeXt-101-FPN | **36.7** | **59.5**  | **38.9**  |

- loss 函数比较

|  loss   |    AP    | AP$_{50}$ | AP$_{75}$ |
| :-----: | :------: | :-------: | :-------: |
| softmax |   24.8   |   44.1    |   25.1    |
| sigmoid | **30.3** | **51.2**  | **31.5**  |

sigmoid + 二值 loss 能避免竞争，提高准确率

- RoIAlign 比较

|   Method   | align？ | bilinear | agg. |    AP    | AP$_{50}$ | AP$_{75}$ |
| :--------: | :----: | :------: | :--: | :------: | :-------: | :-------: |
| RoIPooling |   no   |    no    | max  |   26.9   |   48.8    |   26.4    |
|  RoIWarp   |   no   |   yes    | max  |   27.2   |   49.2    |   27.1    |
|  RoIWarp   |   no   |   yes    | ave  |   27.1   |   48.9    |   27.1    |
|  RoIAlign  |  yes   |   yes    | max  | **30.2** | **51.0**  | **31.8**  |
|  RoIAlign  |  yes   |   yes    | ave  | **30.3** | **51.2**  | **31.5**  |

RoIAlign 提高 segmentation 准确率；max / avg 的集合方法没有大区别

- mask branch 比较

|      |               mask branch                |    AP    | AP$_{50}$ | AP$_{75}$ |
| :--: | :--------------------------------------: | :------: | :-------: | :-------: |
| MLP  |  fc: $1024 \to 1024 \to 80 \cdot 28^2$   |   31.5   |   53.7    |   32.8    |
| MLP  | fc: $1024 \to 1024 \to 1024 \to 80\cdot 28^2$ |   31.5   |   54.0    |   32.6    |
| FCN  | conv: $256\to 256\to 256\to 256\to 256\to 80$ | **33.6** | **55.2**  | **35.3**  |

FCN 保留空间位置信息，segmentation 结果更好。

###### 参考文献 

1. K. He, G. Gkioxari, P. Dollar, R. Girshick. Mask R-CNN. 