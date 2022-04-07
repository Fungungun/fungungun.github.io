# DSP-SLAM: Object Oriented SLAM with Deep Shape Priors
* 介绍
    * **object-aware** SLAM, 重建以物体为中心的地图，把所有low level的几何原始信息group到属于同一个物体的instance中
    * 单目和立体视觉使用 ORB-SLAM2 作为backbone以及重建的3D点云来重建检测到的物体
    * 立体视觉+lidar的系统使用 stereo orb-slam2 作为 slam backbone然后使用额外的稀疏的lidar测量集来重建物体并做pose-only 优化

* 系统 overview
    * DSP-SLAM 是一个序列性的定位和建图方法，能够重建检测到的物体的全部详细形状的同时用一个稀疏的特征点集来粗略表示背景
    * 每个物体使用一个compact and optimizable code vector $\bf{z}$ 来表示
    * 使用 DeepSDF 作为 shape embedding，输入是一个 shape code $\bf{z} \in \mathbb{R}^{64}$ 和 一个 3D query location $\bf{x} \in \mathbb{R}^{3}$, 输出是一个在给定point的有向距离函数（SDF）的值
    * 稀疏SLAM backbone
        * ORB-SLAM2 用作追踪和建图的backbone，这是一个基于特征的slam架构，可以用于单目和立体sequences。追踪现成估计相机pose，建图线程重建3d landmarks的稀疏地图
    * 检测
        * 在 key frames 做检测和分割，位姿的初始估计通过3d框检测得到（SMOKE和SECOND）
    * Data Association
        * 新的detection 或者被归于一个已知的物体中，或者新建一个物体，每个instance包括一个2D框，一个2D mask， 3d 点云的深度和初始物体pose
    * Prior-based 物体重建
        * 对于新的物体进行重建，takes 3d point observations which can come from reconstructed SLAM points (in monocular and stereo modes) or LiDAR input (in stereo + LiDAR mode) and optimises the shape code and object pose to minimise 表面一致性和深度渲染losses，
        * 对与已经存在于map中的物体只更新其 6-DoF pose
    * 联合地图优化
        * a joint 因子图 of point features(from SLAM), 物体，相机pose is optimised via BA, 来维持一个一致的map和incorporate 闭环。新物体被添加为因子图的nodes并且他们的相对pose 被添加为因子图的 camera-object edges

* 带有shape先验的物体重建
    * 表面一致性 term
        * 这一项测量观察到的3d点和重建的物体表面之间的alignment，理想情况下，3d 点投影回到2d点之后应该完美地和物体表面align，相应SDF的值应该为0.实际中，在只有部分观测的时候，仅仅依靠表面一致项对于正确的形状和位姿估计是不够的，会导致估计出的形状远远大于实际尺寸。因此提出另一个rendering loss
    * 可微分 SDF 渲染器
        * 参考 faceobook的 DeepSDF
        * The SDF value encodes the probability that a given point is occupied by the object or belongs to free space （这一部分要再参考DeepSDF）

* Object SLAM
    * object data association
        * aim to associate each detection to its ***nearest*** object in the map
        * when LiDAR is available, 计算3d框和重建物体之间的距离
        * when only stereo or monocular images are used， 计数检测和物体之间matched feature points 的个数
        * 如果多个detection都和一个物体匹配上，保留 the nearest one 
        * 没有被分配的detections会初始化新的物体并进行重建
        * 对于monocular 和 stereo，只有足够的表面点被观测到之后才会进行重建
        * 分配到的detections，只有pose会被优化，同时一条新的camear-object边被加入到因子图中
    * 联合bundle adjustment
        * joint map 包括 相机pose set，object poses 和地图点，联合BA是一个non-linear least squares 优化问题
        * Camear-Object measurement
        * camera-point measurement

* 实验结果
    * KITTI3D. 和 toyota research 的那篇autolabeling进行比较
    * 3D物体重建
        * 实验设定：使用bev ap 和 distance threshold metric （NS）作为评估metric
    * 结果
        * dsp-slam is better ,more accurate shape and pose 
    * 时间分析
        * 为了实时的性能，使用高斯牛顿求解器，更快收敛
    * 使用250个点重建的结果和50个点差不多，但是50个点减少到10个点重建退化比较明显
