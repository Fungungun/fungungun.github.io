# AutoShape: Real-Time Shape-Aware Monocular 3D Object Detection 

* 介绍
    * 自动驾驶中感知系统中，感知周围障碍物的3D形状和位姿是一个基本的任务
    * Lidar expansive. 因此单目的3d物体检测很重要
    * 单目方法中的主要挑战是获取准确的深度信息。有方法使用深度估计获取pseudo lidar 然后通过lidar detection方法获取3d物体的位姿，但是计算量大
    * 直接方法：SMOKE 使用一个中心点来代表物体，因此忽视了详细的外观信息
    * RTM3D 引入了8个关键点（3d 框的8个角点），但是这些关键点没有真实的环境意义并且对应的2d关键点随着相机的运动和自身pose改变变化剧烈
* 问题定义
    * 姿态估计
        * 估计物体的3D旋转和平移 $(\mathbf{R}, \mathbf{T})$
    * learning-based 3D 物体检测
        * 直接回归所有参数
* 提出的方法
    * 点级别的2D-3D约束
        * 大部分情况下路面都是平的，因此角度从三个减少到一个，只保留yaw
        * 通过关键点以及投影关系，运算得到解
        * 2d关键点有的会被遮挡，采用输出一个score作为关键点的权重
        * 手动标贵，因此使用之歌自动标注的pipeline来优化2D 3D的重投影误差
    * 网络
        * 使用DLS-34作为backbone，类似CenterNet是one stage的, 输入图像 W x H, 输出是原来的四分之一
        * 物体中心branch 
            * classification 输出 $\frac{W}{4} \times \frac{H}{4} \times C$ where $C$ is the number of classes
            * regression 输出 $\frac{W}{4} \times \frac{H}{4} \times 2$ 代表 x,y 的offset
        * 物体dimension
            * h,w,l : $\frac{W}{4} \times \frac{H}{4} \times 3$
            * 不输出绝对尺寸，而是输出相对于每个class mean 尺寸的相对尺寸
        * 2D 关键点
            * 回归相对于中心点的offset : $\frac{W}{4} \times \frac{H}{4} \times 2n$
        * 3D 关键点
            * 回归相对于中心点的offset : $\frac{W}{4} \times \frac{H}{4} \times 3n$
        * 物体旋转
            * Multi-Bin baaed method [31], 8 bins, $\frac{W}{4} \times \frac{H}{4} \times 8$
        * 关键点置信度
            $\frac{W}{4} \times \frac{H}{4} \times 2n$
        * 3D IOU score
    * Loss函数
        * 所有loss加权和
* 3D 形状 自动标注
    * 采用一个3D可变性车辆模板可以通过调整参数来表示任意车辆形状，因此，3D shape标注过程可以被抽象为一个优化问题，找到一个shape参数能最优fit视觉观察（2d mask， 3d box， 3d lidar points）
    * 使用一组PCA basis作为模板
    * 获取标注数据，2d mask， 3d box 和 3d lidar points，对于3d lidar points使用ransac移除地面点
    * 已有的benchmark只提供yaw角，实验发现另外两个角度可以大幅度提高标注结果
    * 2d loss： mask distance，像素级插值
    * 3d loss： 对每一个点找到最近的点然后计算距离
    * 使用梯度下降进行优化
* 实验结果
    * 。。。
