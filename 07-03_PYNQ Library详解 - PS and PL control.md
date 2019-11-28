# 前言

PYNQ也提供overlay的一些底层控制，包括overlay的控制和管理，以及PL的底层控制等，包括：

## **PS control**

- PMBus

## **PL control**

- Overlay
- PL and Bitstream classes

------



# **PMBus**

PYNQ 提供了电压和当前板上提供的许多使用PMBus（或其他LINUX核支持的协议）的传感器的访问权限。PYNQ使用libsensors API (<https://github.com/lm-sensors/lm-sensors>)来监控传感器。

- ### pynq.pmbus API

所有的传感器可以通过使用pynq.get_rails()函数找到，该函数返回一个字典，标记了电压轨道名字到一个 Rail 类。每一个 Rail 类有成员voltage,  current 和 power 传感器，当前状态读取可以从value 属性得到。

- ### The DataRecorder

PMBus库的另一个方面是提供了能在一次测试里一个记录多个传感器值的DataRecorder 类。一个DataRecorder 是由被监控的传感器构造的，并最终会产生一个pandas DataFrame作为结果。record(sample_interval) 函数开始以指定的采样速率进行记录。stop() 函数会停止记录。如果 record 函数在with 块里使用，那么stop 会在块的最后自动调用，以确保监控线程是始终结束的。每一个样本由时间戳进行索引编号并包含一个会话识别器。识别器从0开始并且随着每一次 record 或者 mark() 的调用而增长。这个识别器允许不同部分可以更进一步的分析。

**实例：**

```javascript
from pynq import get_rails, DataRecorder
 
rails = get_rails()
recorder = DataRecorder(rails['12V'].power)
 
with recorder.record(0.2): # Sample every 200 ms
    # Perform the first part of the test
    recorder.mark()
    # Perform the second part of the test
 
results = recorder.frame
```

# **Overlay**

Overlay类是用来加载PYNQ overlays到PL上，并且管理控制已有的overlays。这个类是由overlay的.BIT文件进行实例化。默认的话，overlay的Tcl文件将会被解析，并且bitstream将会下载到PL上。这意味着要用overlay类，.BIT和.TCL必须得有。

若要不加载.BIT文件就实例化overlay，那就在实例化的时候传入参数download=False。

一开始下载bitstream后，overlay .TCL文件的时钟设定就会在bitstream下载完毕前被应用。

**实例：**

.BIT文件路径可以是相对的也可以是绝对。

```javascript
from pynq import Overlay
 
base = Overlay("base.bit") # bitstream implicitly downloaded to PL
 
base = Overlay("base.bit", download=False) # Overlay is instantiated, but bitstream is not downloaded to PL
 
base.download() # Explicitly download bitstream to PL
 
base.is_loaded() # Checks if a bitstream is loaded
 
base.reset() # Resets all the dictionaries kept int he overlay
 
base.load_ip_data(myIP, data) # Provides a function to write data to the memory space of an IP
                              # data is assumed to be in binary format
```

ip_dict 包含了overlay的一列IP，并可以用来决定IP驱动、物理地址、版本、或者连接到IP上的中断。

```javascript
base.ip_dict
```

# **PL and Bitstream classes**

PL主要是一个Overlay类使用的内部类。PL类创立一个PL server来管理下载好的overlay。PL server能够防止不同应用的多个overlays覆写当前加载的overlay。

Overlay Tcl文件是由PL类解析并生成IP、clock、中断、gpio字典（IP、clocks和overlay信号的相关信息）。

Bitstream类可以在pl.py里找到，并可以用来下载一个bitstream文件到PL上而不需要使用overlay Tcl文件。这可以用来调试，但是这些属性也可以从Overlay类里获取到。使用Overlay类来访问这些属性是推荐的方法。

**实例：**

- ### PL


```javascript
PL.timestamp # Get the timestamp when the current overlay was loaded
 
PL.ip_dict # List IP in the overlay
 
PL.gpio_dict # List GPIO in the overlay
 
PL.interrupt_controllers # List interrupt controllers in the overlay
 
PL.interrupt_pins # List interrupt pins in the overlay
 
PL.hierarchy_dict # List the hierarchies in the overlay
```

- ### Bitstream


```javascript
from pynq import Bitstream
 
bit = Bitstream("base.bit") # No overlay Tcl file required
 
bit.download()
 
bit.bitfile_name
```

‘/opt/python3.6/lib/python3.6/site-packages/pynq/overlays/base/base.bit’

更多有关PL模块信息可以在pynq.pl模块找到。