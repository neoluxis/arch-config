# OpenMV 使用 AprilTag

##  什么是 AprilTag

**AprilTag**是一种视觉基准系统，可用于多种任务，包括增强现实、机器人和相机校准。可以通过普通打印机创建目标，AprilTag 检测软件可以计算标签相对于相机的精确 3D 位置、方向和标识。AprilTag 库是用 C 实现的，没有外部依赖项。它旨在轻松包含在其他应用程序中，并可移植到嵌入式设备。即使在手机级处理器上也可以实现实时性能。 

只要把这个tag贴到目标上，就可以在OpenMV上识别出这个标签的3D位置，id。

它大概长这样：

![apriltag-1](./openmv-apriltag.assets/image-20230712102636833-1689128798021-1.png)

[April Tag 网站：https://april.eecs.umich.edu/software/apriltag.html](https://april.eecs.umich.edu/software/apriltag.html)

## AprilTag的种类

AprilTag的种类叫家族（family）,有下面的几种：

> TAG16H5 → 0 to 29
> TAG25H7 → 0 to 241
> TAG25H9 → 0 to 34
> TAG36H10 → 0 to 2319
> TAG36H11 → 0 to 586
> ARTOOLKIT → 0 to 511

也就是说TAG16H5的家族（family）有30个，每一个都有对应的id，从0~29。

那么不同的家族，有什么区别呢？

比如说TAG16H5的有效区域是4 x 4的方块，那么它比TAG36H11看的更远（因为他有6 x 6个方块）。但是TAG16H5的错误率比TAG36H11高很多，因为TAG36H11的校验信息多，所以，如果没有别的理由，**推荐用TAG36H11**。



##  制作AprilTag

很简单，你可以在网络上下载，也可以直接从OpenMV的IDE里生成。 在工具——Machine Vision——AprilTag Generate中选择family，推荐使用TAG36H11。

填写需要生成的个数

![image-20230712103110874](./openmv-apriltag.assets/image-20230712103110874.png)

然后选择一下图片保存位置

![image-20230712103206776](./openmv-apriltag.assets/image-20230712103206776.png)

打开文件夹查看：

![image-20230712103244173](./openmv-apriltag.assets/image-20230712103244173.png)

## 基本识别

```python
# AprilTags Example
#
# This example shows the power of the OpenMV Cam to detect April Tags
# on the OpenMV Cam M7. The M4 versions cannot detect April Tags.

import sensor, image, time, math

sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QQVGA) # we run out of memory if the resolution is much bigger...
sensor.skip_frames(30)
sensor.set_auto_gain(False)  # must turn this off to prevent image washout...
sensor.set_auto_whitebal(False)  # must turn this off to prevent image washout...
clock = time.clock()

while(True):
    clock.tick()
    img = sensor.snapshot()
    for tag in img.find_apriltags(): # defaults to TAG36H11 without "families".
        img.draw_rectangle(tag.rect(), color = (255, 0, 0))
        img.draw_cross(tag.cx(), tag.cy(), color = (0, 255, 0))
        degress = 180 * tag.rotation() / math.pi
        print(tag.id(),degress)

```

在这段代码中，传感器拍摄一张照片，

## 3D 位置
