# PYNQ中文资料

------

Xilinx作为芯片生产商，正在不断地推出各种隐藏底层硬件细节的软件开发工具链，希望藉此提高软件开发者的生产效率，让硬件开发者做好硬件开发的工作，软件开发者只需在硬件开发者的基础上继续构建，而无需面面俱到的了解底层的实现原理，HLS和SDSoC工具就是其中的代表。HLS使得工程师可以快速将算法在C/C++级别进行硬件化加速，省去了HDL调试与优化的巨大精力。SDSoC使得工程师可以快速抉择将模块部署在可编程逻辑PL部分还是处理器PS部分，从而在最短的时间内调整到最优的系统性能。

尽管如此，HLS和SDSoC还是需要工程师对FPGA开发流程有较深的了解，开发应用的C/C++语言在易用性和可读性上依旧有所欠缺。正因如此，Xilinx推出的Pynq开发框架，结合了简单易学易上手的Python语言，上层应用开发者可以真正摆脱底层硬件细节的纠缠，将性能瓶颈交给专业的硬件工程师，专心开发纯软件层面的应用。

PYNQ作为一种全新的框架，很多同学和工程师都还比较模式，中文资料相对也比较欠缺，基于此，我们计划逐渐为大家增加更多的中文资料，帮助大家尽早熟悉PYNQ框架。

# 内容列表

------

- PYNQ-Z2开发板上手
- PYNQ框架介绍
- Jupyter Notebook必知必会
- PYNQ Overlay介绍
- BaseOverlay介绍
- Logictools Overlay
- PYNQ Library详解 - PS与PL接口
- PYNQ Library详解 - IP访问
- PYNQ Library详解 - PS and PL control
- PYNQ Library详解 - IOP
- PYNQ Library详解 - Pynq MicroBlaze
- PYNQ快速上手实验介绍
- Overlay设计方法学
- 自定义Overlay设计流程
- 基于HLS的加速器Overlay设计实例 - 快速生成硬件IP
- 基于HLS的加速器Overlay设计实例 - Notebook中调用硬件IP
- 第三方Overlay介绍-SPYN
- 以BNN-PYNQ为例的自定义Overlay分发方法介绍
- Python基础