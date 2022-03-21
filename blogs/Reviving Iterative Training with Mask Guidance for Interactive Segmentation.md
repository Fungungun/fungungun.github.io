# Reviving Iterative Training with Mask Guidance for Interactive Segmentation

* 方法
    * 修改网络结构
        * 交互式分割网络结构和实例分割与语义分割很相似，都有高分辨率输入和输出，主要区别是用户输入的编码和处理，因此无需重新设计网络结构，可以使用SOTA的分割网络然后侧重于交互的部分
        * DeepLabV3+ 被研究比较多，HRNet+OCR比较新而且比较promising，实验结果HRNet效果更好
        * 点击编码：distance transform 和 disks 两种编码方式，实验结果disks效果更好 
            * 对于disk编码来说，添加新的点击只造成local的影响，DT编码在添加新的点的时候变化剧烈，会confuse网络
        * 编码后的点击如何使用网络处理
            * 一般都是扩大第一层卷积层使得网络可以接收N通道的输入而不只是RGB图像，例如Distance Maps Fusion (DMF) 模块，可以将图像与用户输入concated一起的输入转成3通道的输入（e.g. 4 channel -> 3 channel)
            * 引入一个新的卷积块，输出的tensor形状和原backbone第一个卷积块的输出tensor形状完全一致，之后会对这两个卷积块的输出求element-wise sum作为后续的输入，这种改进可以为新的卷积块设置单独的学习率而不影响预训练网络的权重

    * 重复采样策略
        * 不直接受用mislabelled region的中心店，而是先对mislabelled region进行形态学腐蚀，这样腐蚀后的面积减小4倍。实验证明选取中心点的操作会对NoC指标过拟合，实际操作时，当用户点击物体边缘或者mislabelled region的地方时，模型的表现会很不稳定或者更差
        * 本文将随机采样和重复采样结合：1. 模拟之前工作的随机采样策略 2. 设置最大迭代次数$N_{iter}$, 从$0$增加到$N_{iter}$个模拟点击

    * 结合前一步的masks
        * 本文模型将mask，编码的positive clicks，编码的negative clicks组成一个三通道输入

    * Normalized Focal Loss
        * 二值交叉熵损失对所有样本一视同仁，导致后期训练速度会变慢, Focal Loss 可以解决这个问题，但是FL的梯度会渐渐消失，NFL可以解决这个问题


* 数据集
    * 现有数据集
        * Semantic Boundaries Dataset, Pascal VOC, 共有10582张图片，25832实例masks，20个类别：7种交通工具，6种动物，6种室内物体，人。类别很有限
        * large scale: OpenImages, LVIS. LVIS: 1.2M instances on 100k images >1000 classes. OpenImages: 2.6M instances on 944k images 350 categories. LVIS 标注质量最高
        * COCO + LVIS. COCO: 1.2M instance masks on 118k images 80 categories. 

