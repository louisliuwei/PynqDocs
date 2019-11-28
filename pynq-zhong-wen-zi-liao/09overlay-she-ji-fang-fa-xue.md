# 09\_Overlay设计方法学

**前言：**如PYNQ介绍里描述的那样，overlays跟软件里的库类似。程序员可以将overlays实时的下载到Zynq® PL为软件应用提供所需的功能。

一个overlay是一个FPGA逻辑设计类。FPGA逻辑设计通常会为特定任务做相应的优化。Overlay是为广泛应用提供可配置和重用的功能而设计。一个PYNQ的overlay通常有Python的接口，让软件工程师像使用其它的Python包一样使用。

软件工程师会使用overlay，但通常不会创建overlay，因为创建overlay需要非常高的硬件设计技能。

创建overlay需要很多的组件：

* Board Settings
* PS-PL Interface
* MicroBlaze Soft Processors
* Python/C Integration
* Python AsyncIO
* Python Overlay API
* Python Packaging

本章节将给出overlay的创建流程，以及如何将overlay集成到PYNQ中，但不会涉及详细的硬件设计细节，对硬件设计者来说必须掌握的技能。

## Overlay设计

Overlay包含两个主要的组件：FPGA逻辑设计（bitstream）以及工程block diagram的tcl文件。Overlay设计是硬件工程师的任务，本章节将假设读者已经具备了使用Vivado工具进行ZYNQ系统构建和开发的能力。

## PL设计

Xilinx® Vivado工具被用于创建Zynq系统，生成用于烧录Zynq PL侧的Bitstream文件。

我们可以到如下链接下载免费的WebPack版本。

[https://www.xilinx.com/products/design-tools/vivado/vivado-webpack.html](https://www.xilinx.com/products/design-tools/vivado/vivado-webpack.html)

鼓励硬件设计者为PYNQ overlay中需要使用的IP提供开发支持。一旦IP设计完成，PL侧的设计就和其它的Zynq系统开发流程一样。一个能被PYNQ使用的IP应该是可以内存映射和GPIO相连的。IP也可以作为AXI主设备。PYNQ提供了各种与PL接口的libraries，可以很方便的使用这些libraries来创建我们自己的驱动。下一节将会讲到PYNQ overlay的API。

## Overlay Tcl文件

Tcl文件在PL设计中，由Vivado IPI模块设计（block design）产生，PYNQ使用Tcl文件来自动识别Zynq系统的配置、IP及版本、中断、复位以及其它的控制信号。基于这些信息，PYNQ可以自动修改某些系统配置、为设备自动分配驱动、使能或者禁用某些系统特性，信号被连接到相应的Python方法。

作为overlay的一部分，Tcl文件必须和bitstream文件一起提供。Tcl文件可在overlay硬件设计完成后，通过导出（exporting）IP Integrator block diagram来产生。Tcl文件应该在下载overlay的时候和bitstream一起被提供，PYNQ PL类会自动分析Tcl文件里的内容。

一个定制化（custom）或者手动创建的Tcl文件可被用于build一个Vivado工程，但是Vivado工具应该被用于根据block diagram产生和导出Tcl文件。这个自动生成的Tcl文件可确保PYNQ可以做正确的分析。

通过Vivado根据图形界面来为Block Design产生Tcl文件：

_Click **File** &gt; **Export** &gt; **Block Design**_

或者，也可以在Tcl console里：

_write\_bd\_tcl_

Tcl文件的名字必须和bitstream的名字匹配，比如my\_overlay.bit和my\_overlay.tcl。

Tcl文件的内容会在overlay实例化和下载的时候被分析。

![](../.gitbook/assets/01%20%281%29.png)

当找不到Tcl文件或者Tcl文件名字与Bitstream文件名字不匹配的时候，系统会报错。

## 可编程性（Programmability）

Overlay应该在bitstream生成完成之后依然具备可编程性，允许对系统进行客制化。大量的可重用的PYNQ IP模块对PYNQ的可编程性提供了支持。比如，Microblaze可被用于Pmod和Arduino接口。来自于不同overlay的可重用的IP，使得PYNQ在运行时具备可配置性。

## Zynq PS配置

基于Zynq器件的Vivado工程具备两个部分：PL设计与PS配置设置。

PYNQ镜像被用于启动板卡及在启动阶段对PS侧进行配置，它覆盖了大多数PS侧需要的配置，包括DRAM配置和使能PS侧的外围设备，比如SD卡、以太网、USB和UART等。

配置包含了PS侧的时钟配置和PL侧使用的时钟配置。PL侧的时钟可根据Overlay的不同需求，在运行时进行配置。该配置由PYNQ Overlay类自动管理。

在下载新的overlay的过程中，会分析Tcl文件获取时钟配置。在下载bitstream之前会首先完成时钟配置。

## 已有的Overlays

已有的overlays可以作为创建新的overlay的起点。Base overlay可以在PYNQ代码库的board目录找到，包括板卡外围设备的参考IP。

_/boards//base_

目录下有现成的makefile文件，可用于重新build Vivado工程生成overlay需要的bitstream文件和Tcl文件。注：在Windows操作系统下，通过source Tcl文件来代替make方法build工程。

已经运行PYNQ框架的板卡上有可用的Base overlay的bitstream和Tcl文件，也可以在PYNQ的github工程目录下找到。

_/boards//base_

## 板卡配置

### Base overlay工程

对应板卡的Base overlay源代码可在PYNQ github上找到。工程可通过makefile或者Tcl文件重新build。

Base overlay可作为新设计一个overlay的起点。

### Vivado板卡配置文件（board files）

Vivado板卡配置文件包含创建一个新的Vivado工程所需要的配置。以下为PYNQ-Z2配置文件的下载地址。

[http://www.tul.com.tw/download/PYNQ-Z2\_board\_file\_v1.0.zip](http://www.tul.com.tw/download/PYNQ-Z2_board_file_v1.0.zip)

将配置文件安装到Vivado工具后，允许在创建Vivado工程的时候选择板卡以使工具对Zynq PS侧进行配置。

解压缩板卡配置文件并拷贝到如下目录完成板卡配置文件的安装。

_\Vivado\\data\boards_

注：如果在拷贝板卡配置文件时Vivado是处于打开状态，则需要重新打开Vivado工具。

### XDC约束文件

PYNQ-Z2的xdc约束文件可在如下地址进行下载：

[http://www.tul.com.tw/download/PYNQ-Z2\_v1.0.xdc.zip](http://www.tul.com.tw/download/PYNQ-Z2_v1.0.xdc.zip)

## PS/PL接口

![](../.gitbook/assets/02%20%287%29.png)

Zynq器件的PS与PL之间有9个AXI总线接口。在PL侧有4个AXI master HP（High performance）接口，2个AXI master GP（General Purpose）接口，2个AXI slave GP接口和1个AXI master ACP接口。在PS侧还有连接到PL侧的GPIO控制器。

总共有4个用于管理PS和PL数据传输的pynq类。

* GPIO - General Purpose Input/Output
* MMIO - Memory Mapped IO
* Xlnk - Memory allocation
* DMA - Direct Memory Access

具体使用哪个类，取决于IP所连接到的Zynq PS接口和IP的接口。

运行在PYNQ上的Python代码可以访问连接到AXI slave GP口的IP。_MMIO_被用于对该类接口进行操作。

连接到AXI master接口的IP不能由PS直接控制。AXI master接口允许IP直接访问DRAM。在这样做之前，必须对IP使用的内存进行分配（allocate）。Xlnx类被用于对该类接口进行操作。

DMA可被用于Zynq PS DRAM与PL之间高性能的数据传输。PYNQ的DMA类提供了对此类接口进行操作。

### PS GPIO

总共由64个从PS到PL侧的GPIO。

从PS到PL侧的GPIO可被用于简单的通讯，比如这些GPIO可被用于复位（Resets）或者中断（Interrupts）。

GPIO并不需要映射到系统内存空间来访问。

更多关于PS GPIO的信息，请查看附件Pynq\_libraries-PS GPIO。

### MMIO

任何连接到AXI slave接口的IP都会被映射到系统内存空间。_MMIO_可被用于读写内存映射的地址。一次MMIO操作会完成从内存空间读取或者存储32位数据。因为MMIO不支持突发方式进行数据的传输，所以MMIO比较适合通过AXI slave接口连接到PS侧的IP之间的小批量数据交互。

更多关于MMIO的信息，请查看附件Pynq\_libraries-PS MMIO。

### Xlnk

在IP访问DRAM空间之前，内存必须被分配。Xlnk类允许对内存空间进行分配。Xlnk会分配一段连续的内存空间，以便于PS与PL之间进行高效的数据传输。Python或者其它运行在Linux上的代码可直接对这段内存进行访问。

因为PYNQ运行在Linux操作系统上，这段内存会存在与Linux的虚拟内存。Zynq AXI master接口的IP可对物理内存（Physical memory）进行访问。Xlnk提供了一个指向物理内存的指针，该指针可以被发送给overlay中的IP。物理地址被存储在了已分配内存空间实例的_physical\_address_属性中。Overlay中的IP可通过该物理地址访问同一段内存空间。

更多关于Xlnk的信息，请查看附件Pynq\_libraries-Xlnk。

### DMA

AXI stream接口通常被用于高性能的流应用（streaming applications）。AXI stream可通过连接到Zynq AXI HP口的DMA使用。

pynq.DMA类支持AXI DMA IP。允许从DRAM读取数据后发送到AXI stream，或者从AXI stream获取数据后写到DRAM。

更多关于DMA的信息，请查看附件Pynq\_libraries-DMA。

### Interrupt

在python环境中，有特定与asyncio events相连的中断。为了集成到PYNQ框架，特定的中断必须被连接到AXI中断控制器，然后再连接到PS的第一个中断线（first interrupt line）。如果有超过32个中断，则可对AXI中断控制器进行级联。这是为了给其它非PYNQ直接控制的中断做预留，比如SDSoC加速器需要用到的中断。

中断由Interrupt类进行管理，基于PYNQ标准库中的asyncio。

更多关于中断类的信息，请查看附件Pynq\_libraries-Interrupt。

更多关于asyncio的信息，请查看Asyncio章节的内容。

## PYNQ Microblaze子系统

## Python-C 集成（Integration）

在有些情况下，通过Python类来管理与overlay的数据传输的效率并不高。通常情况下，性能由Python驱动和更高的性能库可以通过使用低级语言（比如C/C++）来开发，并可针对overlay进行相应的优化。库中的驱动函数可以从Python里通过CFFI（C foreign Function Interface）进行调用。

CFFI提供了一个C代码与Python之间的简单接口方式。CFFI包已经预安装到了PYNQ镜像中，它支持，API和ABI模式，这两种模式又分别包含“in-line”和“out-of-line”编译模式。Inline ABI\(Application Binary Interface\)兼容模式允许动态加载和运行来自与可执行模块的函数，API模式允许编译C扩展模块。

下面是一个在Python种调用 C函数strlen\(\)的例子。

C函数的原型如下：

```javascript
size_t strlen(const char\*);
```

_C函数原型被传递给_cdef\(\)，然后通过clib调用。

```javascript
from cffi import FFI
ffi = FFI()
ffi.cdef("size_t strlen(const char*);")
clib = ffi.dlopen(None)
length = clib.strlen(b"String to be evaluated.")
print("{}".format(length))
```

​ 在公共库里的C函数可以在Python里通过CFFI调用。公共库可以使用CFFI在线编译，也可以离线编译。

​ 关于更多关于CFFI和公共库的信息可以参考：

[http://cffi.readthedocs.io/en/latest/overview.html](http://cffi.readthedocs.io/en/latest/overview.html)

[http://www.tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html](http://www.tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html)

#### PYNQ and Asyncio

与硬件交互经常会遇见等待加速器完结束或者等待数据。Polling不是一个有效的方式来等待数据，尤其是Python这种同时只能有一个线程在执行的语言来说。

Python asyncio库可以异步的管理多个IO-bound任务，因而可以避免来自等待响应或者低速IO子系统的阻塞。相反的，程序可以继续去执行其它准备好运行的任务。当之前繁忙的任务可以恢复后，它们会继续执行，循环会重复。

在PYNQ里，实时任务会经常使用PL侧的IP block来实现。同时，这些在PL侧执行的任务在任何时刻都有可能向PS侧发送中断。Python的asyncio库提供了一种管理这类来自于异步、IO-bound任务事件的有效方式。

PYNQ中的pynq.interrupt模块中的Interrupts类是asyncio的基础，它提供了一种可被用于等待中断产生的aysncio类事件。Video库，AXI GPIO和PynqMicroblaze驱动都是基于中断事件，提供了用于任何可能阻塞的函数的协程（coroutine）。

#### Asyncio基本原理

Asyncio 并发框架基于协程（coroutine），futures, 任务（tasks）和一个事件循环。在通过简单的例子阐明它们的使用方法之前，将会对他们做一下简要的介绍。

**协程（Coroutines）**

协程是一种新的Python语言结构（Construct）。协程引入了两个关键字，await和async。协程是可被暂停的状态函数（Stateful Functions）。这意味着当它们在等待任务或者事件完成的时候可退出执行。当被暂停，协程会保持它的状态。当等待的任务或者事件结束后协程会恢复运行。await关键字决定了协程中程序停止和恢复执行的位置。

#### Futures

Futures是一个代理或者一个对象的包装，不是真实的目标对象。一旦异步计算完成，你就可以提取它。Futures是asyncio的重要组件：它可以将中止的操作封装后放入队列中，可查询它的完成状态，当操作完成后可获取返回结果，例化过程由并发框架例化，不需要用户直接干预。

#### 任务（Tasks）

协程并不直接执行，它们被封装成任务（tasks），并注册为事件循环（event loop）。Tasks是futures的子类。

#### 事件循环（Event loop）

事件循环负责执行所有准备好的任务，采用轮询的方式执行。

一个事件循环一次只执行一个任务，它依赖于协同调度，这意味着任务之间不会相互影响，当任务阻塞运行时会将控制权交给事件循环，这将会导致单线程并发代码中的所有的事件处理函数都按顺序执行后才会执行下一次循环。

下面是一个简单的例子，通过async def关键字定义了一个命名为wake\_up的协程。函数主要将wake\_up协程封装为wake\_up\_task任务并将其注册到事件循环中。在协程中，关键字await标注了程序被挂起和恢复的位置。事件循环执行了下面的任务。

1. 开始执行start\_up\_task任务；
2. 将start\_up\_task任务挂起并记录它的状态；
3. ​ 运行asyncio.sleep函数1到5秒钟；
4. 根据保存的状态恢复wake\_up\_task函数运行；
5. 运行任务直至结束，任务关闭。

最终事件循环关闭。

示例运行完毕后会有如下输出：

![](../.gitbook/assets/03%20%283%29.png)

在事件循环中的任何阻塞调用应该用协程来代替。如果不这么做，当阻塞调用发生时，它会阻塞循环的其它部分的运行。

如果你需要用到阻塞调用，应该将其放在单独的线程里。计算负载较大的任务也应该放在单独的线程/进程中。

### Pynq Asyncio实例

Asyncio可被用于管理Overlay中各种阻塞操作。一个协程可以被运行在事件序列中来等待中断事件的发生。其它的用户程序也可以运行在事件循环中。如果中断发生，任何等待的协程将会被重新调度。中断协程的响应性取决于用户代码交出线程控制权的频率。

#### GPIO外围设备

用户I/O设备可能会产生中断，比如按钮被按下或者开关状态改变。Pynq的Button和Switch类中都有一个wait\_for\_level的成员函数和一个wait\_for\_level\_async的协程，它们都会被阻塞，直到按钮或者开关返回切换到特定的状态。Pynq中规定package中的所有协程都带有**\_async**后缀。

我们以Led灯控制为例，当按钮按下时对应的Led灯会亮起。首先需要定义一个含有该功能的协程。

![](../.gitbook/assets/04%20%284%29.png)

 然后将协程注册到默认的事件循环中。

![](../.gitbook/assets/05%20%281%29.png)

最后，运行已经注册好协程的事件循环。

![](../.gitbook/assets/06%20%285%29.png)

#### PynqMicroblaze

在PynqMicroblaze中有一个Interrupt的成员变量，功能和asyncio类似。事件带有一个wait\(\)协程和一个clear\(\)方法。事件会被自动连接到正确的中断管脚，如果加载的Overlay中没有中断则会返回None。

例如：

![](../.gitbook/assets/07%20%286%29.png)

有两种方法来运行新的IOP封装类中的函数-从外部的asyncio事件循环中调用或者建立自己的事件循环然后从事件循环中来调用asyncio函数。

#### AsyncIO函数

对于中断驱动的功能，Pynq中同时提供了asyncio协程和阻塞函数的调用的方法。用户自定义的驱动也推荐提供这样的2种调用方法。阻塞函数可被用于不需要协程或者接受阻塞出现的事件循环中。

以下是一段定义协程的代码，可以注意到只需要添加async和await关键字就可以把一个函数定义为一个asyncio协程。

![](../.gitbook/assets/08%20%281%29.png)

#### 事件循环（Event Loops）

以下的代码实现了对asyncio协程封装，注册到默认的事件循环中，运行协程直到结束。

![](../.gitbook/assets/09%20%284%29.png)

#### 客制化的中断处理（Custom Interrupt Handling）

Interrupt类允许客制化中断处理。

这个类对PL侧的AXI中断控制器管理进行了抽象化，我们并不需要详细的了解代码的细节。该中断类利用了中断连线的pin名字，为其提供了一个wait\_async协程和相应的wait函数。中断

以下是使用中断的通用模式：

![](../.gitbook/assets/10%20%285%29.png)

该模式避免了同一个中断被使用多次的竞争危害。

#### Examples

更多的例子请参考AsyncIO Buttons的notebook，可以在如下目录找到：

![](../.gitbook/assets/11%20%284%29.png)

