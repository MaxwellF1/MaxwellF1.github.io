# CUDA Environment Configuration


## Windows

在Windows上安装CUDA只需要根据官网上的指南安装对应的版本并添加到系统环境变量里就可以正常使用了。但是需要配合Visual Studio，感觉这个软件过于庞大繁杂。

### 命令行设置

希望能够从头开始熟悉CUDA的编写、编译流程等，希望能够不用Visual Studio而是可以像Linux下直接通过命令行编译.cu文件并生成可执行文件，通过查询，根据[1](http://iliutong.cn/2019/01/20/nvcc-cu-file-in-console-in-windows/)完成了该配置。配置过程如下：

首先要将VS的VC编译器放在环境变量中，对于我的64位系统而言要将用于x64的VC编译器放在环境变量中：

1. 在系统变量Path中添加如下的两个条目：

   ```shell
   C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64
   C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE
   ```

2. 在系统变量中新建一个叫LIB的变量，并为其添加三个条目：

   ```shell
   C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\lib\x64
   C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\ucrt\x64
   C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\um\x64
   ```

3. 在系统变量中新建一个叫INCLUDE的变量，为其添加两个条目。

   ```shell
   C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.16.27023\include
   C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\ucrt
   ```

   以上三个步骤的作用其实是让 VC 编译器脱离 Visual Studio IDE 环境生效，也就是说可以让 VC 编译器 cl.exe 在命令行下编译 .cpp 或 .c 文件。因为 nvcc 编译器虽说是 .cu 的编译器，但是它还是要调用 VC 编译器 cl.exe 来对 .cu 文件进行编译，这也就是说为什么 CUDA 离不开 Visual Studio，而在其他平台上，比如 Linux，可能 CUDA 需要调用 gcc 或 g++ 编译器来完成对 .cu 文件的编译。经过这个配置过程后就可以通过nvcc指令加上相关参数来编译执行代码了。(比如我现在的thinkpadt470p需要加上-arch=compute_35的选项，因为硬件和默认的编译选项不匹配)。



## WSL

由于想用到linux相关的命令行，而且觉得从头开始编写和编译能够理解的更清楚，但电脑上又没有双系统。我从CUDA的官方文档中看到了对WSL的支持，就去网上搜索了相关的教程。一开始安装好了CUDA，但是nvidia-smi无法访问显卡，以为是WSL设置的原因，搞了好久。最后又重读了一遍CUDA的官方文档发现对Windows系统的版本有要求，最好是20145以上的，但我查看自己的windows10系统，居然只是19xxx，而且检查过后居然没有可用的更新。

后来发现必须加入Windows预览体验计划才能更新这些版本，自己加入并选择DEV Channel后居然自动给我更新了win11...只能硬着头皮上了，感觉自己的电脑配置不一定够用，安好之后又按照官方文档重来一遍，这次可以正常在WSL访问到GPU并使用CUDA了。不得不说win11长得属实有点像MACOS啊，而且这次警示我一定不要先瞎去网上找资料，**RTFM**。



## 一些问题

1. **明明安装好了CUDA，却无法使用NVCC的相关命令：**

   这通常是没有配置好环境变量导致的。

   例如在Linux系统下，可以打开~/.bahsrc，在其中添加环境变量

   ```bash
   export LD_LIBRARY_PATH=/usr/local/cuda/lib
   export PATH=$PATH:/usr/local/cuda/bin
   ```

   并执行`source ~/.bashrc`使得环境变量生效。

   

2. 



## 参考文献

1. [Windows 下在命令行编译 CUDA 文件](http://iliutong.cn/2019/01/20/nvcc-cu-file-in-console-in-windows/)
2. 

