# Pynq hands-on demos介绍

PYNQ-Z2快速上手demo集锦，清单如下所示

- ComputerVision
- DeepLearning
- InternetOfThings

###### 安装

下载整个项目的压缩包([链接](https://github.com/xupsh/pynq-hands-on-demos/archive/master.zip))，并将它复制（可以通过网络、离线等多种方式）到你的PYNQ-Z2上。

每个文件夹中都有单独的使用指南（离线）。

------



# cv2PYNQ

这是一个在PYNQ平台上加速OpenCV图像处理算法的Python扩展包。这个库目前实现了某几个特定的图像处理算法的硬件加速，可以在16ms内处理完1080p的灰度图的滤波算法。

目前已经实现的算法列表：

- Sobel: 3x3; 5x5
- Scharr
- Laplacian: ksize = 1; 3; 5
- blur: ksize = 3
- GaussinBlur: ksize = 3
- erode: ksize = 3
- dilate: ksize = 3
- Canny

###### 安装

取决于Pynq软件版本的不同，安装方式各有不同。

首先需要打开PYNQ-Z1/Z2板卡上的Linux命令行界面，然后根据不同版本输入如下安装命令：

> > = PYNQ v2.3

```javascript
sudo pip3 install -e .
```

> <= PYNQ v2.2

```
sudo pip3.6 install -e . 
```

运行完安装脚本之后就可以在Jupyter界面看到`cv2PYNQ`的文件夹

###### 运行Sobel滤波算法的案例

在`cv2PYNQ`文件夹中有一个Sobel滤波算法的notebook，跟着其中的步骤做就可以了。

------



# BNN-PYNQ

这个项目实现了在PYNQ上部署量化神经网络的任务，目前实现了多种不同精度的量化网络结构：

- 1 bit weights and 1 bit activation (W1A1) for CNV and LFC
- 1 bit weights and 2 bit activation (W1A2) for CNV and LFC
- 2 bit weights and 2 bit activation (W2A2) for CNV

###### 安装

取决于Pynq软件版本的不同，安装方式各有不同。

首先需要打开PYNQ-Z1/Z2板卡上的Linux命令行界面，然后根据不同版本输入如下安装命令：

> > = PYNQ v2.3

```javascript
sudo pip3 install -e .
```

> <= PYNQ v2.2

```javascript
sudo pip3.6 install -e . 
```

运行完安装脚本之后就可以在Jupyter界面看到`bnn`的文件夹

###### 运行Road-Signs路标识别

在`bnn`文件夹中有一个路标识别的notebook,跟着其中的步骤做就可以了。

------



# IoT

这个demo会教你如何在IoT场景中控制传感器和制动器

###### 安装

打开Jupyter首页，将如下两个notebook文件上传到Jupyter中即可。

- `arduino_grove_ledbar.ipynb`
- `pmod_grove_usranger.ipynb`
- `ledbar_and_ultrasonic_ranger.ipynb`

###### 运行Demo

准备物件：

- `Base Arduino shield` <http://wiki.seeedstudio.com/Base_Shield_V2/>
- `Pmod grove adapter` <https://store.digilentinc.com/pynq-grove-system-add-on-board/>
- grove led bar <http://wiki.seeedstudio.com/Grove-LED_Bar/>
- grove ultrasonic ranger sensor <http://wiki.seeedstudio.com/Grove-Ultrasonic_Ranger/>

打开刚刚上传的notebook，根据其中的指令一步步照做即可。

以**超声波测距仪传感器**这个Demo为例。

------



# 超声波测距仪传感器

通过Jupyter打开*InternetOfThings*目录下的pmod_grove_usranger.ipynb

这个例子展示了如何使用 [超声波测距仪传感器](https://www.seeedstudio.com/Grove---Ultrasonic-Ranger-p-960.html)。它的测量最大范围为400cm，测量最小范围是3cm，分辨率为1cm。

如果没有障碍物，则会默认返回500cm。

在这个notebook里，我们只展示如何控制grove ultrasonic ranger连接到Pmod接口上，因此需要一个pmod grove和转换器。当然读者也可以自己把控制移植到Arduino接口的版本上去。

[![usranger](https://github.com/xupsh/pynq-hands-on-demos/raw/master/InternetOfThings/2.png)](https://github.com/xupsh/pynq-hands-on-demos/blob/master/InternetOfThings/2.png)

```javascript
from pynq.overlays.base import BaseOverlay

base = BaseOverlay("base.bit")
```

- ### 使用 Microblaze 去控制超声波传感器

下面的程序假设超声波传感器是连接在Pmod-Grove转接器的G1接口上的，以及该转接器连接在PMODA接口上。

时钟控制器的寄存器分布如下：

| Register name | Register functionality              | Register value |
| ------------- | ----------------------------------- | -------------- |
| TCSR0         | Timer 0 Control and Status Register | 0x00           |
| TLR0          | Timer 0 Load Register               | 0x04           |
| TCR0          | Timer 0 Counter Register            | 0x08           |
| TCSR1         | Timer 1 Control and Status Register | 0x10           |
| TLR1          | Timer 1 Load Register               | 0x14           |
| TCR1          | Timer 1 Counter Register            | 0x18           |

```javascript
%%microblaze base.PMODA

#include "xparameters.h"
#include "xtmrctr.h"
#include "gpio.h"
#include "timer.h"
#include <pmod_grove.h>

#define TCSR0 0x00
#define TLR0 0x04
#define TCR0 0x08
#define TCSR1 0x10
#define TLR1 0x14
#define TCR1 0x18
#define MAX_COUNT 0xFFFFFFFF

void create_10us_pulse(gpio usranger){
    gpio_set_direction(usranger, GPIO_OUT);
    gpio_write(usranger, 0);
    delay_us(2);
    gpio_write(usranger, 1);
    delay_us(10);
    gpio_write(usranger, 0);
}

void configure_as_input(gpio usranger){
    gpio_set_direction(usranger, GPIO_IN);
}

unsigned int capture_duration(gpio usranger){
    unsigned int count1, count2;
    count1=0;
    count2=0;
    XTmrCtr_WriteReg(XPAR_TMRCTR_0_BASEADDR, 0, TLR0, 0x0);
    XTmrCtr_WriteReg(XPAR_TMRCTR_0_BASEADDR, 0, TCSR0, 0x190);
    while(!gpio_read(usranger));
    count1=XTmrCtr_ReadReg(XPAR_TMRCTR_0_BASEADDR, 0, TCR0);
    while(gpio_read(usranger));
    count2=XTmrCtr_ReadReg(XPAR_TMRCTR_0_BASEADDR, 0, TCR0);
    if(count2 > count1) {
        return (count2 - count1);
    } else {
        return((MAX_COUNT - count1) + count2);  
    }
}

unsigned int read_raw(){
    gpio usranger;
    usranger = gpio_open(PMOD_G1_A);
    create_10us_pulse(usranger);
    configure_as_input(usranger);
    return capture_duration(usranger);
}
```

- ### 测量距离

记住放一些障碍物在传感器面前，否则它将返回默认的500cm。

```javascript
from pynq import Clocks

def read_distance_cm():
    raw_value = read_raw()
    clk_period_ns = int(1000 / Clocks.fclk0_mhz)
    num_microseconds = raw_value * clk_period_ns * 0.001
    if num_microseconds * 0.001 > 30:
        return 500
    else:
        return num_microseconds/58
read_distance_cm()
11.873448275862069
```