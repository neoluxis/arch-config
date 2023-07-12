# OpenMV 使用 AprilTag

##  什么是 AprilTag

**AprilTag**是一种视觉基准系统，可用于多种任务，包括增强现实、机器人和相机校准。可以通过普通打印机创建目标，AprilTag 检测软件可以计算标签相对于相机的精确 3D 位置、方向和标识。AprilTag 库是用 C 实现的，没有外部依赖项。它旨在轻松包含在其他应用程序中，并可移植到嵌入式设备。即使在手机级处理器上也可以实现实时性能。 

只要把这个tag贴到目标上，就可以在OpenMV上识别出这个标签的3D位置，id。

它大概长这样：

![apriltag-1](./openmv-apriltag.assets/image-20230712102636833-1689128798021-1.png)

[April Tag]()
