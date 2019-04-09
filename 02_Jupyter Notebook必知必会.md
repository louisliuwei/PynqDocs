## 前言

本节将介绍在PYNQ开发中常见的Jupyter Notebook相关知识。如果想要获取更为完整的Jupyter Notebook的使用技巧可以访问Jupyter官网<http://jupyter.org/>。

------



### Jupyter Notebook简介

Jupyter是基于网页的用于交互计算的应用程序，可被应用于全过程计算：开发、文档编写、运行代码和展示结果。Jupyter可以很方便地部署在嵌入式Linux中，作为嵌入式Linux的Web IDE。

Notebook主要由两部分组成，网页应用和文档部分。网页应用即基于网页形式的、结合了编写说明文档、数学公式、交互计算和其他富媒体形式的工具。简言之，网页应用是可以实现各种功能的工具。至于文档，Notebook中所有交互计算、编写说明文档、数学公式、图片以及其他富媒体形式的输入和输出，都是以文档的形式体现的。这些文档是保存为后缀名为.ipynb的JSON格式文件，不仅便于版本控制，也方便与他人共享。此外，文档还可以导出为：HTML、LaTeX、PDF等格式。

Notebook的下列特点使得它在可视化编程与展示、编写教程文档等方面应用极为广泛。

- 编程时具有语法高亮、缩进、tab补全的功能。

- 可直接通过浏览器运行代码，同时在代码块下方展示运行结果。

- 以富媒体格式展示计算结果。富媒体格式包括：HTML，LaTeX，PNG，SVG等。

- 对代码编写说明文档或语句时，支持Markdown语法。

- 支持使用LaTeX编写数学性说明。

------



###  用户UI界面

这一节主要介绍的是用户UI界面中各个部分的功能组件。当我们输入密码xilinx之后看到的第一页就是Notebook的Dashboard，左侧上方Files代表着当前显示目录文件树，可以看到是否有正在运行的Notebook。在Running页面可以看到当前所有正在运行的Notebook的总览。在Nbextensions页面可以选择添加或删除不同的Notebook插件。Dashboard的右上方的两个按钮的功能分别是上传新文件和创建新文件/文件夹。

<p align="center">
<img src ="images/Chapter_02/Jupyter_UI.png">
</p>
<p align = "center">
<i></i>
</p>

双击打开要编辑的Notebook之后，就进入了Notebook的编辑模式。页面最上方的是该Notebook的文件名，文件名右侧显示上一次保存的相关信息。第二行是Notebook的菜单栏，对Notebook的所有操作都可以在菜单栏的下拉菜单中找到。第二行右侧的是状态栏，常见状态如正在初始化、当前空闲、正在执行当前单元格的代码、运行死机都会显示在这里。第三行是快捷操作按钮，罗列出一些常用的快捷按钮（当然也可以用键盘快捷键代替）。中部是Notebook的正文，Notebook以单元格Cell为最小单位进行编排，每个单元格可以是代码单元格，也可以是Markdown单元格。点击上方的运行按钮即可运行该单元格的代码。Markdown单元格运行时将遵循Markdown语法对源文本进行渲染，常用来写备注和注释。代码单元格中存放的是可运行的代码，在本书中这里指的都是python代码，代码运行的输出直接附在该单元格的下方。

代码单元格左侧[n]内的数字代表着这是该Notebook第n次运行代码单元格，而在运行过程中会显示为[\*]，代表该单元格正在运行。在Notebook中，一次只能运行一个单元格，运行序号靠后的单元格必须在序号考前的单元格运行完毕后才能轮到运行，在此之前只能显示[\*]或者上一次运行的编号。同一个Notebook中将共享一份全局变量表，运行序号靠前的代码单元格中的变量可以被靠后的代码单元格使用。

<p align="center">
<img src ="images/Chapter_02/JuputerMenu.png">
</p>
<p align = "center">
<i></i>
</p>

想要了解更多关于Jupyter的知识，可以通过chrome浏览器连接到板卡后打开getting_started目录的的notebooks动手操作。
<p align="center">
<img src ="images/Chapter_02/Jupyter_Env.PNG">
</p>
<p align = "center">
<i></i>
</p>
