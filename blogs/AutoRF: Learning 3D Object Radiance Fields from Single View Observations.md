# AutoRF: Learning 3D Object Radiance Fields from Single View Observations
* 方法
    * Given a single input image, our goal is to encode each 3D object that is present in the scene into a compact representation that allows to, e.g., efficiently store the objects into a database and re-synthesize them from different views/contexts in a later stage.
    * 不需要物体几何先验知识：CAD 模型，对称等
    * 使用预训练的实例或者全景分割找到同一物体的2d像素点，使用预训练的单目3d检测器找到物体3d空间的pose，训练和测试时，每张图可以获取一系列3d框和对应的2dmask，还有相机标定信息
    * 预先准备
        * 图像：先进行单目3d和分割
        * 归一化物体坐标空间：将3dbox进行归一化
        * 以物体为中心的相机
        * occypancy mask： 前景1，背景-1，遮挡或者无法确定0
    * 架构overview
        * 输入：图像，归一化物体坐标空间中的相机（derived by 挖掘物体3d框和mask的信息）
        * 形状和外观编码器：CNN，提出的feature送到平行的两个heads，分别用来生成shape和appearance的code
        * 形状解码器：shape code 被送到解码器网络，可以隐式输出occupancy network，给定一个NOCS里面的3d点可以输出density
        * 外观解码器：同时输入shape和appearance codes，给定3d点和viewing direction输出rgb颜色
        * volume rendering 体渲染 【26】： NERF
    * 训练
        * photometric loss:The difference between the predicted color of the pixel  and the actual color of the pixel makes the photometric loss.
        * occupancy loss
    * Test time optimization
