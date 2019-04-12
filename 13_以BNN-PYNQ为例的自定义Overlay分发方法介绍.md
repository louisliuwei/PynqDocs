# 如何将一个完成的FPGA工程转换为PYNQ第三方包

Python 有非常丰富的第三方库可以使用，很多PYNQ开发者也会在 Github 上提交自己的适用于PYNQ的 Python 包。将一个完成的FPGA工程转换为PYNQ第三方包会方便我们进行PYNQ的开发。这个过程主要包括两个步骤：1.通过已经在PYNQ里的APIs对FPGA部分进行驱动的编写。2.将编写好的驱动做成Python库并且进行打包分发。

<p align="center">
<img src ="images\Chapter_13\01.png">
</p>
<p align = "center">
<i></i>
</p>

从PYNQ框架中可以看到，对于FPGA Bitstreams中的内容，PYNQ有overlays方法对于Bitstream进行类似Vivado中download Bitstream的行为。对于FPGA中其余部分，比如User designs里的内容，可以将它们作为PYNQ的IPs，通过上层的API（如GPIO、MMIO等）对它进行自定义的驱动编写。或者通过其他方法（如：通过python调用C来实现已经在HLS中实现的算法等）。

本节以BNN-PYNQ工程为例，探究在BNN-PYNQ中如何将一个完成的FPGA工程转换为PYNQ第三方包。

首先为第一个步骤，即对FPGA部分进行驱动的编写。在示例的notebook中，首先导入了bnn这个包，在bnn.py这个.py文件中有对bnn的详细描述，其中便包括了对FPGA部分的驱动编写。在bnn.py中，主要包含对三个类的定义：PynqBNN、CnvClassifier、LfcClassifier。其中，PynqBNN主要作为一个共享库加载指定网络并且将bitstream下载到FPGA（PL）中；CnvClassifier是CNV网络的分类器类，用于对cifar10格式的图像和需要预处理的图像进行推理；LfcClassifier是LFC网络的分类器类，用于对mnist格式的图像进行推理。两个类在构造时，都会加载共享库，并且将bitstream下载到FPGA（PL）中。如在CnvClassifier中构造函数的定义：

   ```javascipt
   def __init__(self, network, params, runtime=RUNTIME_HW):
	if params in available_params(network):
		self.net = network
		self.params = params
		self.runtime = runtime
		self.usecPerImage = 0.0
		self.bnn = PynqBNN(runtime, network)
		self.bnn.load_parameters(os.path.join(params, network))
		self.classes = self.bnn.classes
	else:
		print("ERROR: parameters are not availlable for {0}".format(network))
   ```

​       在CnvClassifier的构造函数中，实例化了一个PynqBNN对象。而在PynqBNN中，构造函数的描述如下：

  ```javascipt
  def __init__(self, runtime, network, load_overlay=True):
	self.bitstream_name = None
	if runtime == RUNTIME_HW:
		self.bitstream_name="{0}-{1}.bit".format(network,PLATFORM)
		self.bitstream_path=os.path.join(BNN_BIT_DIR, self.bitstream_name)
		if PL.bitfile_name != self.bitstream_path:
			if load_overlay:
				Overlay(self.bitstream_path).download()
			else:
				raise RuntimeError("Incorrect Overlay loaded")
	dllname = "{0}-{1}-{2}.so".format(runtime, network,PLATFORM)
	if dllname not in _libraries:
		_libraries[dllname] = _ffi.dlopen(os.path.join(BNN_LIB_DIR, dllname))
	self.interface = _libraries[dllname]
	self.num_classes = 0
  ```

​       在PynqBNN的构造函数中，主要部分即为通过os进行文件检索等操作找到bitstream，并且使用Overlay加载它。因此在示例中实例化一个CnvClassifier对象时，PYNQ板卡上的Done信号灯会闪烁（即代表有bitstream加载到PL部分）。

​       在BNN-PYNQ中，因为网络的相关定义主要通过Vivado HLS进行编写，主要语言对象为C++。因此，项目的核心用法为将C++代码编译为shared library后，python主要作为一个接口对象通过C++共享库与主机库进行通信，具体方法为借助CFFI调用C动态库。如在bnn.py中对CFFI的使用：

   ```javascipt
   import cffi
_ffi = cffi.FFI()

_ffi.cdef("""
void load_parameters(const char* path);
int inference(const char* path, int results[64], int number_class, float *usecPerImage);
int* inference_multiple(const char* path, int number_class, int *image_number, float *usecPerImage, int enable_detail);
void free_results(int * result);
void deinit();
   ```
在通过源文件编译及在python声明之后，就可以通过python调用c语言定义的函数了，如在PynqBNN中对网络的参数进行读取的函数的定义：

  ```javascipt 
  def load_parameters(self, params):
	if not os.path.isabs(params):
		params = os.path.join(BNN_PARAM_DIR, params)
	if os.path.isdir(params):
		self.interface.load_parameters(params.encode())
		self.classes = []
		with open (os.path.join(params, "classes.txt")) as f:
			self.classes = [c.strip() for c in f.readlines()]
			filter(None, self.classes)
	else:
		print("\nERROR: No such parameter directory \"" + params + "\"")

  ```

在确认参数文件路径正确之后，会调用之前的声明的C函数load_parameters，用于加载和读取参数。对工程的更多实现方法及内部原理，可以参考项目主页：

<https://github.com/Xilinx/BNN-PYNQ>    

至此，已经简略了解了BNN-PYNQ的基本工作原理和对FPGA进行驱动编写驱动的过程。

在对驱动编写好后，如果要想向 Github仓库提交自己开发的包，首先要将自己的代码打包，才能上传分发。Python 库打包分发的关键在于编写 setup.py 。setup.py是setuptools的构建脚本。它告诉setuptools你的包（例如名称和版本）以及要包含的代码文件。一个简单的setup.py可以编写如下：

   ```javascipt
    import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="example-pkg-your-username",
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)

   ```

可以从实例中看到，setup.py 文件编写的规则是从 setuptools导入 setup 函数，并传入各类参数进行调用。setup函数常用的参数如下：

| 参数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| name                 | 包名称                                                       |
| version              | 包版本                                                       |
| author               | 程序的作者                                                   |
| author_email         | 作者邮箱                                                     |
| url                  | 程序的官网地址 ，对于许多项目这是一个指向GitHub，GitLab，Bitbucket或类似代码托管服务的链接 |
| description          | 对于程序的简单描述                                           |
| requires             | 指定依赖的其他包                                             |
| packages             | 需要处理的包目录(通常为包含   __init__.py 的文件夹)，对于复杂的工程，可以使用find_packages()去自动发现所有的包 |
| classifiers          | 程序的所属分类列表                                           |
| include_package_data | 自动包含包内所有受版本控制(cvs/svn/git)的数据文件            |
| package_data         | 指定包内需要包含的数据文件                                   |
| data_files           | 打包时需要打包的数据文件，如图片，配置文件等                 |



​       关于setup.py的更多信息，可以参考：

<https://packaging.python.org/guides/distributing-packages-using-setuptools/> 

packages是setup.py中一个重要的参数，它声明了需要处理的包目录。因此在对setup.py进行编写时，需要注意文件的目录结构，BNN的文件目录结构如下图所示：

   ![](images\Chapter_13\02.png)

图5-2          bnn的文件结构

其中，bnn包含了对LfcClassifier和CnvClassifier两个Pyhton类的描述、notebooks中包含了示例notebook，在安装的过程中移到“/home/xilinx/jupyter_notebooks/bnn/”目录下、tests中包含了测试脚本和测试的图片。

BNN的setup.py描述如下（部分核心代码）：

 ```javascipt
 from setuptools import setup, find_packages
import subprocess
import sys
import shutil
import bnn
import os
from glob import glob
import site 

if os.environ['BOARD'] != 'Ultra96' and os.environ['BOARD'] != 'Pynq-Z1' and os.environ['BOARD'] != 'Pynq-Z2':
	print("Only supported on a Ultra96, Pynq-Z1 or Pynq-Z2 Board")
	exit(1)

setup(
	name = "bnn-pynq",
	version = bnn.__version__,
	url = 'kwa/pynq',
	license = 'Apache Software License',
	author = "Nicholas Fraser, Giulio Gambardella, Peter Ogden, Yaman Umuroglu, Christoph Doehring",
	author_email = "pynq_support@xilinx.com",
	include_package_data = True,
	packages = ['bnn'],
	package_data = {
	'' : ['*.bit','*.tcl','*.so','*.bin','*.txt', '*.cpp', '*.h', '*.sh'],
	},
	data_files = [(os.path.join('/home/xilinx/jupyter_notebooks/bnn',root.replace('notebooks/','')), [os.path.join(root, f) for f in files]) for root, dirs, files in os.walk('notebooks/')],
	description = "Classification using a hardware accelerated neural network with different precision for weights and activation"
)
 ```

在此**setup.py**中，主要包括了以下参数：name、version、url、license、author、author_email、include_package_data、packages、package_data、data_files、description。

packages中直接指定了包目录，即bnn。

package_data 中指定了包内需要包含的数据文件，包括：'*.bit'（bit文件，用于比特流的下载）、'*.tcl'（脚本文件，用于解析硬件设计中的ip核信息）、'*.bin'（二进制文件，包含网络训练的参数信息）、'*.cpp', '*.h'（在Vivado HLS中对网络定义时的c++文件和头文件）、'*.so'（c进行编译之后的文件，可以借助cffi调用这个动态链接库）。

data_files中指定了包含的数据路径。

在对PYNQ的setup.py编写时，经常会进行板卡环境的检查，即:

  ```javascipt
  if os.environ['BOARD'] != 'Ultra96' and os.environ['BOARD'] != 'Pynq-Z1' and os.environ['BOARD'] != 'Pynq-Z2':
	print("Only supported on a Ultra96, Pynq-Z1 or Pynq-Z2 Board")
  ```

通过导入os，可以检查板载的详细信息，当板卡不适配时，可以输出错误信息。

在编写好后setup.py后，就完成了对库的打包过程，这时可以选择上传到代码管理网站如Github后，通过pip和git指令安装：

 ```javascipt
sudo pip3 install   git+https://github.com/Xilinx/BNN-PYNQ.git (on PYNQ v2.3)   
 ```

也可以将BNN包源文件下载到PYNQ后，在根目录中，通过pip指令安装：
 ```javascipt
   sudo pip3 install -e . (on PYNQ v2.3)   
 ```