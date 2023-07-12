# OpenMV 使用 AprilTag

##  什么是 AprilTag

**AprilTag**是一种视觉基准系统，可用于多种任务，包括增强现实、机器人和相机校准。可以通过普通打印机创建目标，AprilTag 检测软件可以计算标签相对于相机的精确 3D 位置、方向和标识。AprilTag 库是用 C 实现的，没有外部依赖项。它旨在轻松包含在其他应用程序中，并可移植到嵌入式设备。即使在手机级处理器上也可以实现实时性能。 

只要把这个tag贴到目标上，就可以在OpenMV上识别出这个标签的3D位置，id。

它大概长这样：

![apriltag-1](./openmv-apriltag.assets/image-20230712102636833-1689128798021-1.png)

[April Tag 网站：https://april.eecs.umich.edu/software/apriltag.html](https://april.eecs.umich.edu/software/apriltag.html)

## AprilTag的种类

AprilTag的种类叫家族（family）,有下面的几种：

TAG16H5 → 0 to 29
TAG25H7 → 0 to 241
TAG25H9 → 0 to 34
TAG36H10 → 0 to 2319
TAG36H11 → 0 to 586
ARTOOLKIT → 0 to 511
也就是说TAG16H5的家族（family）有30个，每一个都有对应的id，从0~29。

那么不同的家族，有什么区别呢？

比如说TAG16H5的有效区域是4 x 4的方块，那么它比TAG36H11看的更远（因为他有6 x 6个方块）。但是TAG16H5的错误率比TAG36H11高很多，因为TAG36H11的校验信息多，所以，如果没有别的理由，**推荐用TAG36H11**。
