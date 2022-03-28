* 摘要
    * 单张图片3D重建实现新视角合成与深度估计，通过NeRF进行多平面图片（MPI)连续深度生成

* 方法
    * 3D 表示方法
        * 平面NeRF
            * 使用透视几何来表示相机视锥，使用planes来代替NeRF中的射线，对于视锥中不同深度的平面进行重建
        * 体渲染
            * 平面神经场从两个方面进行离散化：1. 视锥包含N个平面 2. 每个平面都简化成一个4通道的图，第四个通道是深度
            * 新视角渲染可以通过单应性warping实现，
    
    * 网络训练
    * RGB视频监督
        * 本质还是单目深度自监督训练
        * 尺度标定
            * 使用COLMAP来进行SfM获取每张图的稀疏点，然后通过在特定深度的预测来预测scale
        * 损失函数 （4项加权）
            * RGB L1 loss，RGB SSIM loss 为了是生成的rgb图match gt的图
            * edge-aware disparity map smoothness loss，用来惩罚那些在原图上很光滑但是在预测的视差图上剧烈的变化
            * optional sparse disparity loss 
    * 和NeRF的联系
        * 优点： 1. 可以泛化到没见过场景，NeRF需要针对每个场景分别训练 2. less network inference （32 VS millions of inferences) 
        * 缺点： 1. 输入只有一张图，不可能重建360度 2. viewing direction不是输入，因此无法生成复杂的依赖视图的效果
    * 和MPI的关系
        * MPI 是MINE的一种特殊形式，可证
    * 和PixelNerf以及GRF的关系
        * MINE直接对源相机视锥建模，PixelNeRF 和GRF都是对整个3D空间进行建模
        * MINE对每个平面进行重建，另外两个是每个ray进行重建
        * MINE优点：高效，视锥重建而不是像素重建
