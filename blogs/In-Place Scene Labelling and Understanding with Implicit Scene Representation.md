* 摘要
    * 将语义和外观还有几何形状结合起来
    * NeRF的内在多视角一致性和顺滑度可以高效通过稀疏标注生成语义，稀疏和有噪音的情况下都有收益

* 方法
    * NeRF
        * $\bf{\sigma}(\bf{x})$ 体密度，只有3d位置，射线$\bf{c}(\bf{x},\bf{d})$ 包括3d位置和视角方向
    * Semantic-NeRF
        * 相比于NeRF多加了一个semantic head做预测，其他基本完全一致
    * 训练
        * photometric loss 和 semantic loss (mutli-class cross entropy)
        * 每个loss又有两部分组成: corase and fine
        * $L = L_{p} + \lambda L_{s}$. 实验证明训练对于$\lambda$取值并不敏感，因此将其设为1
