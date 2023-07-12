# Open MV 使用 April Tag

##  什么是 April Tag

**April Tag**是一种视觉基准系统，可用于多种任务，包括增强现实、机器人和相机校准。可以通过普通打印机创建目标，April Tag 检测软件可以计算标签相对于相机的精确 3D 位置、方向和标识。April Tag 库是用 C 实现的，没有外部依赖项。它旨在轻松包含在其他应用程序中，并可移植到嵌入式设备。即使在手机级处理器上也可以实现实时性能。 

只要把这个tag贴到目标上，就可以在Open MV上识别出这个标签的3D位置，id。

它大概长这样：

![apriltag-1](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121122104.png)

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

很简单，你可以在网络上下载，也可以直接从Open MV的IDE里生成。 在工具——Machine Vision——April Tag Generate中选择family，推荐使用TAG36H11。

填写需要生成的个数

![image-20230712103110874](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121123205.png)

然后选择一下图片保存位置

![image-20230712103206776](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121123039.png)

打开文件夹查看：

![image-20230712103244173](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121123124.png)

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

在这段代码中，传感器拍摄一张照片，之后在图片中寻找AprilTag，如果找到了就框出来，并在中心划十字。打印出Tag的编号和坐标。

如图：

![image-20230712110818904](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121123319.png)

## 3D 位置

April Tag 具有3D定位的功能，它可以得知Tag的空间位置，一共有6个自由度，三个位置，三个角度。

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

# 注意！与find_qrcodes不同，find_apriltags 不需要软件矫正畸变就可以工作。

f_x = (2.8 / 3.984) * 160 # 默认值
f_y = (2.8 / 2.952) * 120 # 默认值
c_x = 160 * 0.5 # 默认值(image.w * 0.5)
c_y = 120 * 0.5 # 默认值(image.h * 0.5)

# 注意，输出的姿态的单位是弧度，可以转换成角度，但是位置的单位是和你的大小有关，需要等比例换算
def degrees(radians):
    '''将弧度转化为角度'''
    return (180 * radians) / math.pi

while(True):
    clock.tick()
    img = sensor.snapshot()
    for tag in img.find_apriltags(fx=f_x, fy=f_y, cx=c_x, cy=c_y): # 默认为TAG36H11
        img.draw_rectangle(tag.rect(), color = (255, 0, 0))
        img.draw_cross(tag.cx(), tag.cy(), color = (0, 255, 0))
        print_args = (tag.x_translation(), tag.y_translation(), tag.z_translation(), \
            degrees(tag.x_rotation()), degrees(tag.y_rotation()), degrees(tag.z_rotation()))
        # 位置的单位是未知的，旋转的单位是角度
        print("Tx: %f, Ty %f, Tz %f, Rx %f, Ry %f, Rz %f" % print_args)
    print(clock.fps())
```

在串口输出为6个变量，Tx, Ty, Tz为空间的3个位置量，Rx,Ry,Rz为三个旋转量。

![image-20230712112541628](https://testingcf.jsdelivr.net/gh/neoluxis/image/arch/202307121126937.png)

在查找Tag时设置参数 

- f_x 是x的像素为单位的焦距。对于标准的OpenMV，应该等于2.8/3.984\*656，这个值是用毫米为单位的焦距除以x方向的感光元件的长度，乘以x方向的感光元件的像素（OV7725）
- f_y 是y的像素为单位的焦距。对于标准的OpenMV，应该等于2.8/2.952\*488，这个值是用毫米为单位的焦距除以y方向的感光元件的长度，乘以y方向的感光元件的像素（OV7725）
- c_x 是图像的x中心位置
- c_y 是图像的y中心位置

### 计算位置

#### 理论

实际April Tag 的尺寸不确定，所以测出距离的单位也不确定，需要进行换算。

测得的 $t_x$ ，实际April Tag距离摄像头X方向上距离是 $x$ ，那么比例 $k_x = \frac{x}{t_x}$ 

同理，

$k_y = \frac{y}{t_y} \\ k_z = \frac{z}{t_z} $ 

测出三个方向上的 $k$ 值后，就可以计算距离
$$
D = \sqrt{(k_x t_x)^2 + (k_y t_y)^2 + (k_z t_z)^2}
$$

#### 代码

PS：在电脑屏幕上，April Tag 的大小为 9.5 cm

首先使用上面的代码，获取到 $k$ 值：
$$
k_x = 3 \\
k_y = 2 \\
k_z = 5 \\
$$
可以写一个函数

```python

def getData(tag):
    k_x = 3
    k_y = 2
    k_z = 5
    ret = {
        'x_translation': 0.0, # 12
        'y_translation': 0.0, # 13
        'z_translation': 0.0, # 14
        'x_rotation': 0.0,    # 15
        'y_rotation': 0.0,    # 16
        'z_rotation': 0.0,    # 17
        'distance': 0.0,      # calculated value
    }
    ret['x_translation'] = tag.x_translation() * k_x
    ret['y_translation'] = tag.y_translation() * k_y
    ret['z_translation'] = tag.z_translation() * k_z
    ret['x_rotation'] = degrees(tag.x_rotation())
    ret['y_rotation'] = degrees(tag.y_rotation())
    ret['z_rotation'] = degrees(tag.z_rotation())
    ret['distance'] = math.sqrt(ret['x_translation']**2 + ret['y_translation']**2 + ret['z_translation']**2)
    return ret
```

将找到的 tag 传入函数

```python
data = getData(tag)
print(data)
```

![image-20230712121220387](./openmv-apriltag.assets/image-20230712121220387.png)

测距相对较准，因为度量不方便，所以在 $K$ 的时候均

