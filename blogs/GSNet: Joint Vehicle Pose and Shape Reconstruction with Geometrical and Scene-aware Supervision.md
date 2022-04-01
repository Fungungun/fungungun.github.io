# GSNet: Joint Vehicle Pose and Shape Reconstruction with Geometrical and Scene-aware Supervision
* 介绍
    * 几何与场景感知网络，从一张街景图片同时估计6DOF pose 和 重建3d car shape。
    * four-way 特征提取 和融合机制，一次前向直接回归6自由度姿态和形状
    * 分治3D重建

* 姿态和形状表示
    * 6自由度姿态，3d平移和3d旋转，平移是相机坐标系下的物体中心坐标，旋转是物体坐标系下的旋转欧拉角
    * 分治形状表示
        * 使用1352 vertices 和2700 faces 的稠密网格来表示车的形状
        * 使用SoftRas把ApolloCar3D里面的模型的转成本文所定义的拓扑形式
        * 为了简化训练，将79个CAD models用K-Means分成4组，对每一个子集，使用PCA学习一个low dimensional shape basis，对于k个vehicle mesh， 使用PCA找到 $n\leq10$维的shape basis，定义shape basis $\overline{\bf{S}}\in\mathbb{R}^{N\times n}$ where $N>>n$
        * inference时，输入实例先进行4分类，然后对每一个cluster预测principle component coefficient。最终的shape是四个mesh的分类分数的加权blended。
        * 这种策略和直接PCA所有mesh可获得更低的形状重建error

* 网略结构设计
    * 分治特征提取和表示
        * 先做2D检测，然后roi pooling提取出每个instance的feature，另一个平行的branch一个Fully Conv Net 输出2d关键点heatmaps和坐标
        * 通过相机内参将2d box转到世界坐标系。
        * 使用apollo的66个关键点，每个关键点表示为$\{x_{k}, y_{k}, v_{k}\}$, $v_{k}$代表是否可见，使用COCO上pretrained的human pose estimation网络初始化，ROI pooling使用FPN作为backbone
    * 融合机制
        * 提取的four-way特征分别被转换成一维表示然后使用先验知识来决定每一个用于何种任务。
        * 对于全局特征点位置和检测框，使用两个FC层把提取到的特征转到high level
        * 对于ROI特征和heatmaps，使用步长为2的sequential卷积操作将spatial size降到1x1，channel数不变
        * 有选择性的使用不同feature应用于不同的任务：
            * translation 主要影响物体的位置和scale，因此这里将ROI 特征、2d关键点特征和box position 特征concatenate起来做regression
            * 在已知3d形状和纹理的前提下，rotation 决定了物体的image appearance，因此使用ROI feature,heatmap feature and keypoint feature作为输入。
            * 对于shape parameters, 使用ROI和heatmap特征
    * 多任务预测
        * 三个平行branch: translation,rotation and shape reconstruction

* 几何与场景感知监督
    * 通过关键点反投影达到几何一致性
        * 将预测到的3d mesh上的semantic keypoint投影到2d，然后和heatmap regression模块给出的结果进行重投影loss 计算
        * 将66个GT mesh上的3d vertex投影到2d，这66个3d点投影的2d点钟有醉倒2d标注neighbors的被选定为对应的3d landmarks
    * 场景感知loss
        * 观察到大部分车都在一个common ground plane上并且不同实例的高度差不多，因此车中心可以假设为co-planar
        * 对于每张图，we locate mesh centers for four randomly-selected instances，三个中心点确定一个平面方程 $ax+by+cz+d=0$ 并且省下的那个车中心坐标定义为$(x_1, y_1, z_1)$, *inter-instance co-planar loss* is defined as $L_{p\_glo}=|ax_1+by_1+cz_1+d|/(a^2+b^2+c^2)^{1/2}$
        * 车的四个轮子组成的平面也应该和地面平行，定义类似的*intra-instance co-planar loss*不过点是从同一个instance上选取
    * 回归损失
        * L2 loss for 3d shape reconstruction: distance between vertices
        * 6 DoF pose: for translation, L1 loss is better than L2, for rotatino the regression loss is defined according to the difference between the GT and the prediction
    * subtype 分类损失
        * 将car instance分成34个subtypes （sedan, minivan suv, etc)
    * 最终loss
        * weighted sum of all the lossed described before

* 实验
    * Mask R-CNN with resnet101 pretrained COCO2017 for object detection and ROI feature extraction
    * detection score threshold 0.3
    * threre are respectively 9, 24, 14, 32 meshs in the four clusters
    
        


