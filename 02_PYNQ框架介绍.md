# PYNQ-提高ZYNQ器件生产力

<p align="center">
<img src ="images/Chapter_02/01.png">
</p>
<p align = "center">
<i></i>
</p>


# Python已经成为全世界使用最广泛的编程语言

<p align="center">
<img src ="images/Chapter_02/02.png">
</p>
<p align = "center">
<i></i>
</p>

# PYNQ-为Xilinx平台引入更多的软件开发者

<p align="center">
<img src ="images/Chapter_02/03.png">
</p>
<p align = "center">
<i></i>
</p>

# 从Jupyter Notebook到Jupyter Lab

<p align="center">
<img src ="images/Chapter_02/04.png">
</p>
<p align = "center">
<i></i>
</p>

# Jupyter Lab运行在ZYNQ-PS侧 （内部的Cortex-A9或者A53 ARM硬核）

<p align="center">
<img src ="images/Chapter_02/05.png">
</p>
<p align = "center">
<i></i>
</p>

# PYNQ系统架构
<p align="center">
<img src ="images/Chapter_02/06.png">
</p>
<p align = "center">
<i></i>
</p>

# PYNQ-基于Ubuntu系统
<p align="center">
<img src ="images/Chapter_02/07.png">
</p>
<p align = "center">
<i></i>
</p>

# Ubuntu系统-更好的软件生态
<p align="center">
<img src ="images/Chapter_02/08.png">
</p>
<p align = "center">
<i></i>
</p>

# 并非都要成为FPGA设计专家
<p align="center">
<img src ="images/Chapter_02/09.png">
</p>
<p align = "center">
<i></i>
</p>


# 丰富的Overlay可供下载
<p align="center">
<img src ="images/Chapter_02/10.png">
</p>
<p align = "center">
<i></i>
</p>

# PYNQ框架已经为PS和PL的接口提供Linux驱动，并封装为Python库
<p align="center">
<img src ="images/Chapter_02/11.png">
</p>
<p align = "center">
<i></i>
</p>

# Python库-以MMIO为例
<p align="center">
<img src ="images/Chapter_02/12.png">
</p>
<p align = "center">
<i></i>
</p>

# 统一的接口赋予PYNQ框架高效率
<p align="center">
<img src ="images/Chapter_02/13.png">
</p>
<p align = "center">
<i></i>
</p>

# 简单的安装方式
<p align="center">
<img src ="images/Chapter_02/14.png">
</p>
<p align = "center">
<i></i>
</p>

# 加载Overlay到Zynq器件
<p align="center">
<img src ="images/Chapter_02/15.png">
</p>
<p align = "center">
<i></i>
</p>

# 赋予硬件开发者全新的验证方式
<p align="center">
<img src ="images/Chapter_02/16.png">
</p>
<p align = "center">
<i></i>
</p>

<p align="center">
<img src ="images/Chapter_02/17.png">
</p>
<p align = "center">
<i></i>
</p>

# Python赋予我们更多的想象
<p align="center">
<img src ="images/Chapter_02/18.png">
</p>
<p align = "center">
<i></i>
</p>

<p align="center">
<img src ="images/Chapter_02/19.png">
</p>
<p align = "center">
<i></i>
</p>

# Numpy数据如何传输到FPGA
<p align="center">
<img src ="images/Chapter_02/20.png">
</p>
<p align = "center">
<i></i>
</p>

# Numpy数据分析的有效工具
<p align="center">
<img src ="images/Chapter_02/21.png">
</p>
<p align = "center">
<i></i>
</p>

# 云-端协同
<p align="center">
<img src ="images/Chapter_02/22.png">
</p>
<p align = "center">
<i></i>
</p>

<p align="center">
<img src ="images/Chapter_02/23.png">
</p>
<p align = "center">
<i></i>
</p>

# 学生竞赛
<p align="center">
<img src ="images/Chapter_02/Contest.png">
</p>
<p align = "center">
<i></i>
</p>

# 移植到其它平台
<p align="center">
<img src ="images/Chapter_02/24.png">
</p>
<p align = "center">
<i></i>
</p>

什么是PYNQ？**

PYNQ是Python On Zynq的缩写，它是一个软件开发框架，指导硬件层、驱动层和应用层之间的接口设计，不是ISE、Vivado、SDSoC这样的IDE工具，更不是Zynq芯片的下一代芯片产品。

<p align="center">
<img src ="images/Chapter_02/framework.png">
</p>
<p align = "center">
<i></i>
</p>

PYNQ框架的设计初衷是通过高层次的封装，将底层硬件FPGA实现细节与上层应用层的使用脱耦，让上层应用开发者通过Python编程就可以调用FPGA模块，不需要懂Verilog/VHDL硬件编程就可以享受FPGA可并行计算、接口可方便扩展和可灵活配置带来的诸多好处。

在ARM A9 CPU上运行的软件包括：

·         载有Jupyter Notebooks设计环境的网络服务器

·         IPython内核和程序包

·         Linux

·         FPGA的基本硬件库和API

 

**如何在PYNQ上开发？**

请参见网页[www.pynq.io](file:///C:/Users/mqiao/Desktop/PYNQ-Z1%20Python%20Productivity%20for%20Zynq/www.pynq.io)。

​                
 也欢迎您加入PYNQ微信公众号，在这里，您将找到可以帮助您开始使用PYNQ的参考资料和应用案例。
<p align="center">
<img src ="images/Chapter_02/PYNQ.png">
</p>
<p align = "center">
<i></i>
</p>

**PYNQ-Z2****是否支持传统开发方式？**

除了支持PYNQ框架，PYNQ-Z2也可以采用传统的ZYNQ开发方式，使用Vivado, SDK, SDSoC等工具进行开发。