* 动态场景深度估计，对应点的深度预测需要输出可信、平滑的3D空间的运动，这一假设被forumalte到一个test time training的框架中，深度估计的CNN和一个场景流的MLP会在输入的video上一起训练
* 方法
    * 简介
        * 问题：输入一段视频，场景中的物体和相机都在移动，目标输出每帧的深度
        * achieved by 包括两个模型的优化框架：深度预测CNN和场景流预测MLP，深度预测模型使用预训练，场景流预测trained from scratch
        * 前处理，计算camera poses 和 两帧之间的光流，flow consistency loss:预测出来的深度和场景流投到2D后要和预先计算出的光流field 对应上 , depth consistency loss:深度和场景流预测要一致
        * 假设：局部线型运动
    * 前处理
        * RGB图先用ORB-SLAM2进行初始位姿估计，之后通过COLMAP中的SfM阶段来生成每一帧的相机位姿和稀疏深度图，对于motion mask可用的视频，使用COLMAP中的mult-view stereo stage来进行更精确的位子估计和更稠密的深度预测
        * 使用RAFT进行前向和后向（第帧和第i+k帧，k=1,2,4,6,8）optical flow计算
        * 之后通过一个MannequinChallenge model 或者 MiDaS v2 输出每一帧的初始深度，这种方法输出的深度有尺度不变形，因此计算了scale = $mean(median(D_{i}^{init}/D_{i}^{sfm}))$,然后将这个scale应用到所有的相机变换上面
        * 使用成对的前后向光流预测一致性找到被遮挡的区域（光流错误的区域），以此生成遮挡mask用于loss计算，所有不一致性大于1的flow都被认为是被遮挡或者不可靠的
    * test time 深度学习和场景流
        * 问题转化为，有预算的光流、位姿等，优化一个预训练、单帧深度预测网络，并同时从头训练一个场景流预测网络，输入是一个3d的点世界坐标，输出是其到下一时刻的场景流
        * 理论上，场景流估计模块是冗余的，可以通过2d稠密匹配和深度估计来进行估计，但是这种估计nosiy and unstable，使用MLP来对场景流隐式建模对于稳定优化过程很重要
        * objective loss：1. 投影后的深度和场景流 与 预计算光流 loss 2. $D_{i}$ 和 $S_{i->j}$ 与 $D_{i+1}$ 一致loss 3. 预测的场景流的光滑度 loss
    * 训练loss
        * 2D一致性：同一点和其在另一帧之间的像素距离
        * 3D一致性
        * 平滑3D运动
