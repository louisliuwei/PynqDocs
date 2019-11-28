# 前言

PYNQ库提供了对子系统Pynq MicroBlaze的支持。它允许我们加载预编译好的应用，并且可以在Jupyter中创建编译新的应用。

接下来，我们按照如下顺序逐一介绍：

- MicroBlaze Subsystem
- MicroBlaze RPC
- MicroBlaze Library

------

 

# **MicroBlaze Subsystem**

PYNQ MicroBlaze子系统可以由PynqMicroblaze类进行管控，这允许我们从Python下载程序，通过执行处理器的重置信号、共享数据内存读写和管理中断来进行操控。

每一个PYNQ MicroBlaze子系统都含有一个IOP（IO处理器），一个IOP定义了一些能被Python控制的交互与动作控制器。现在一共有三个IOP：Arduino，PMOD，Logictools。

该子系统含有一个MicroBlaze处理器，AXI交互、中断控制器，一个中断请求器和外部系统接口以及BRAM、内存控制器。

AXI交互控制器把MicroBlaze连接到中断控制器、中断请求器和外部接口上。

- 中断控制器是其他连接到MicroBlaze处理器上的交互/动作控制器的接口。

- 中断请求器发送中断请求至Zynq处理系统。

- 外部接口允许MIcroBlaze子系统与其他控制器或DDR内存进行交互。

- BRAM保存了MicroBlaze的指令和数据。

BRAM是双端口的：一个端口连接到MicroBlaze指令与数据端口，另一个连接到ARM® Cortex®-A9 通讯处理器。

如果外部接口连接到了DDR内存，则DDR可以用来在子系统和PS之间传输大量数据分段。

<p align="center">
<img src ="images/Chapter_07/52.png">
</p>
<p align = "center">
<i></i>
</p>

**实例：**

在Base Overlay里，有三个IOP实例可用：PMODA, PMODB, Arduino。
```javascript
from pynq.overlays.base import BaseOverlay
from pynq.lib import PynqMicroblaze
base = BaseOverlay('base.bit')
mb = PynqMicroblaze(base.iop1.mb_info,
 "/home/xilinx/pynq/lib/pmod/pmod_timer.bin")
mb.reset()
```

更多有关PynqMicroblaze类的信息和API可以在pynq.lib.pynqmicroblaze.pynqmicroblaze模块找到。



# 创建一个新的PYNQ MicroBlaze

任何一个包含了MicroBlaze并且满足之前提及的AXI-代码内存、PS中断线、重置线的要求的**视图**（hierarchy）就可以在PYNQ里使用。然而，为了通过IPython magic来使用MicroBlaze，你必须提供一个**电板支持套件**（BSP）。BSPs是由Xilinx SDK里生成，并包含了所有的连接到MicroBlaze的外设的驱动与配置信息。

PYNQ提供了TCL脚本来从硬件描述文件中生成BSP，我们可以在boards/sw_repo下找到，与之一起的还有Python或者C语言需要的驱动和pynqmb硬件概览库。创建并使用BSP需要以下几个步骤：

- 从Vivado输出Hardware来生成HDF文件

- 在boards/sw_repo文件夹下运行指令make HDF=$HDF_FILE。如果没有提供HDF，那么Base Overlay的BSPs就会被生成。

- 复制生成了的BSP到板上并确保名字为bsp_${hierarchy}，这里${hierarchy}是MicroBlaze视图的名字。如果你有多个MicroBlaze，对于所有的视图我们可以采用相同的前缀。

- 在Python里调用add_bsp(BSP_DIR)。这是初始化你自己的overlay的一部分操作。

这些步骤会把BSP整合到PYNQ里并允许IPython在新的MicroBlaze子系统下使用*%%MicroBlaze*魔术。如果你希望重新使用一个已有的子系统，只要它的命名与Base Overlay一致——例如iop_pmod* 或者iop_arduino*，那么正确的BSP就会被自动使用。



# MicroBlaze RPC

PYNQ MicroBlaze基础设备是构建在**远程调用**（RPC）层上的，该层是负责发送函数调用指令到MicroBlaze并处理所有的数据传送。

远程调用层支持用于接口函数的C语言。任何不满足以下要求的函数将会被略过：

- 传递参数与返回的内容里没有struct类或者union类

- 返回内容不能是指针类型

- 不出现指针的指针。

所有的返回值都是通过复制的方式传回给Python。函数参数的传递依赖参数使用的类型。对于给定的非空原始数据按如下法则：

- 非指针类型从PYNQ复制到MicroBlaze

- 常指针类型从Python复制到MicroBlaze

- 非常指针类型从Python复制到MicroBlaze，然后再在函数完毕后复制回去。

函数执行的顺序可以从下面看出：

Python struct模块是用来把传递给函数的Python类型转换成MicroBlaze使用的合适的整数或者浮点值。在转换域以外的那些值会导致异常，使得MicroBlaze函数无法运行。数组类型与struct的处理方法类似，从Python的数组类转成C数组。对于非常数数组，数组会在适当的地方更新使得返回值能被调用者读取。转换规则的唯一异常就是char和const char指针，这俩会被优化成Python的bytearray和bytes类型。注意对一个bytes对象使用一个非常数char*变量调用一个函数会导致错误因为bytes对象是只读的。

- ## **Long-running Functions**

对于返回为非空的函数，Python函数是同步的，它会等到前面的C函数结束后才返回到调用者。对于那些返回为空的函数，则函数是被异步调用的，Python函数会立即返回。这就牵扯到那些运行时间很长却又相互独立的函数如何在不堵塞Python线程的情况下载MicroBlaze上运行。当函数在运行的时候，其他函数无法调用除非这个运行时间很长的过程不停的调用yield来允许RPC处理请求。注意！MicroBlaze里是不支持多线程的，因此想要同时运行两个长时间过程只会导致最后只运行了其中一个，即便使用了yield功能。

- ## **Typedefs**

RPC引擎完全支持typedefs并提供了额外的机制来使得C函数更像Python类。RPC层会识别那些typedef名作为前缀的函数名。举一个例子，i2c typedef有函数i2c_read和i2c_write，他们都以i2c类型作为第一个参数。于是RPC会创建一个类叫做i2c并有read和write方法。若想要达成这种转换，必须满足下述性质：

- Typedef是一个原始type

- 至少有一个函数会返回这个typedef

- 至少一个函数命名遵循这个模式。

 

# **MicroBlaze Library**

PYNQ MicroBlaze库是与MicroBlaze子系统交互的主要方法。它含有一列包装好了的IO控制器的驱动，并对连接到PYNQ I/O开关的情况作了优化。

这篇文档描述了所有的C函数和类。

- ## General Principles

这个库提供了GPIO,I2C,SPI,PWM/Timer和UART函数。所有的这些库都遵循相同的设计。每一个都定义了一个类型，代表了设备的一个句柄。*_open函数是用在设计了IO开关并用了引脚连接设备的情形下。引脚的数目取决于协议。*_open_device打开一个特定的设备并可以传递控制器的地址或者索引（由BSP定义）。*_close是用来释放一个句柄。

 

- ## GPIO Devices

GPIO设备允许一个或多个引脚被直接读写。所有的函数都在gpio.h里。

### gpio type

一个对单引脚或多引脚的句柄。

- gpio` `gpio_open(int` `pin)

根据特定的IO开关上的引脚返回给GPIO设备一个新的句柄。这个函数只能在设计中有一个IO开关的时候被调用。

- gpio` `gpio_open_device(unsigned` `int` `device)

返回给AXI GPIO处理器一个句柄，基于基地址或者设备索引。这个句柄允许通道1上的所有引脚被同时设定。

- gpio` `gpio_configure(gpio` `parent,` `int` `low,` `int` `hi,` `int` `channel)

返回一个绑定到控制器特定引脚的新句柄。这个函数不会改变父句柄的配置。

- void` `gpio_set_direction(gpio` `device,` `int` `direction)

设定特定句柄的所有引脚的方向。方向只能是GPIO_IN 或 GPIO_OUT.

- void` `gpio_write(gpio` `device,` `unsigned` `int` `value)

设定句柄的输出引脚的值。如果该句柄有多引脚，那么最不重要的位会指代最低索引引脚。往已经配置好的引脚写东西没有任何作用。

- unsigned` `int` `gpio_read(gpio` `device)

读取句柄的输入引脚的值，若果有多引脚，则最不重要的位将会指代最低索引引脚。从配置好的引脚读取只会读取到0。

- void` `gpio_close(gpio_device)

将特定引脚重置到高阻抗输出并关闭设备。



- ## **I2C Devices**

I2C驱动仅为master操作设计，并提供接口从slave device中进行读取。所有的这些函数都在i2c.h里。

- i2c type

代表了一个I2C master。多句柄都指代相同的master设备是允许的。

- i2c` `i2c_open(int` `sda,` `int` `scl)

打开一个连接上了IO开关I2C设备（IO开关已经配置为使用特定引脚）。调用这个函数会断开先前分配好的引脚并将它们重置到高阻抗状态。

- i2c` `i2c_open_device(unsigned` `int` `device)

通过基地址或者ID打开一个I2C master。

- void` `i2c_read(i2c` `dev_id,` `unsigned` `int` `slave_address,` `unsigned` `char*` `buffer,` `unsigned` `int` `length)

生成一个读指令到特定的slave。 buffer 是一个数组，由长度不小于 length的调用者分配。

- void` `i2c_write(i2c` `dev_id,` `unsigned` `int` `slave_address,` `unsigned` `char*` `buffer,` `unsigned` `int` `length)

生成一个写指令到特定的slave。

- void` `i2c_close(i2c` `dev_id)

关闭I2C设备。



- ## **SPI Devices**

SPI操作数据的同步传输，因此，没有读与写操作，只有传输函数。这些函数在spi.h里。

###### spi type

一个SPI master的句柄。

- spi` `spi_open(unsigned` `int` `spiclk,` `unsigned` `int` `miso,` `unsigned` `int` `mosi,` `unsigned` `int` `ss)

在特定引脚上打开一个SPI master。如果一个设备不需要引脚，那么传入“-1”即可。

- spi` `spi_open_device(unsigned` `int` `device)

根据基地址或者ID打开一个SPI master。

- spi` `spi_configure(spi` `dev_id,` `unsigned` `int` `clk_phase,` `unsigned` `int` `clk_polarity)

使用特定的clock 相位与极性配置SPI master。这些设定对于一个SPI master的所有句柄都是通用的。

- void` `spi_transfer(spi` `dev_id,` `const` `char*` `write_data,` `char*` `read_data,` `unsigned` `int` `length);

从或者往SPI slave传输字节。write_data 和 write_data 应该是由调用者或者NULL分配的。缓冲的最小长度为length.

- void` `spi_close(spi` `dev_id)

关闭一个SPI master



- ## **Timer Devices**

计时设备有两个作用。他们既可以被用来输出PWM信号也可以作为程序计时器来插入精确地延迟。同时使用上述两个功能是不可能的，如果在PWM正在运行时而又要去尝试延迟，则会导致undefined问题。所有的这些函数都在timer.h里。

### timer type

一个AXI计时器句柄。

- timer` `timer_open(unsigned` `int` `pin)

打开一个连接到特定引脚的AXI计时器。

- timer` `timer_open_device(unsigned` `int` `device)

用地址或者设备ID打开计时器。

- void` `timer_delay(timer` `dev_id,` `unsigned` `int` `cycles)

用指定的一段时间来进行延迟。

- void` `timer_pwm_generate(timer` `dev_id,` `unsigned` `int` `period,` `unsigned` `int` `pulse)

用指定的计时器生成PWM信号。

- void ` `timer_pwm_stop(timer` `dev_id)

停止PWM输出。

- void` `timer_close(timer` `dev_id)

关闭指定的计时器。

- void` `delay_us(unsigned` `int` `us)

用微秒数来延迟程序，使用默认延迟计时器（编号0）。

- void` `delay_ms(unsigned` `int` `ms)

用毫秒数来延迟程序，使用默认延迟计时器（编号0）。



- ## **UART Devices**

这个设备驱动控制一个UART master。

- uart ` `type

一个UART master设备的句柄。

- uart` `uart_open(unsigned` `int` `tx,` `unsigned` `int` `rx)

在特定引脚上打开UART设备。

- uart` `uart_open_device(unsigned` `int` `device)

用基地址或者索引打开UART设备。

- void` `uart_read(uart` `dev_id,` `char*` `read_data,` `unsigned` `int` `length)

从UART读取固定长度的数据。

- void uart_write(uart dev_id, char* write_data, unsigned int length)

写一段数据到UART。

- void  uart_close(uart` `dev_id)

关闭UART。
