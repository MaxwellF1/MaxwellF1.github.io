# Udacity CS344-Intro to Parallel Programming-Unit2-GPU Hardware and Parallel Communication Patterns


对于新的高性能架构，通常能源效益应该放在设计的第一位，之后要紧接考虑的是可编程性和应用范围。(performance goal: extremscale, 10^18)。

## Communication Patterns

并行计算都是关于很多线程协同工作来解决问题，关键是协同工作，而协同工作与通信有关，在CUDA中，通信主要发生在内存相关。

### Map and Gather

Map：对每一个数据进行同样的函数或计算任务。即每个任务在内存中特定位置读取和写入，输出输出之间是**一对一**的关系。

Gather: 每个线程计算并存储一系列元素的值，并输出一个计算结果，即**多->一/多**，例如求平均值。

### Scatter

Tasks compute where to write output。可以理解为Gather的逆操作，每个并行任务需要将结果写入不同的地方或多个地方，而要写入哪个地方是该任务计算得出的。

### Stencil(模板)

Tasks read input from a fixed neighborhood in an array. <u>Data Reuse!</u> 

以固定的格式(模板)来更新元素。例如：2D冯诺依曼模板，2D Moore pattern。相邻操作会有重叠，即数据重用，很多线程访问并计算相同的数据。每个元素被访问的数量与模板中元素数量相同，例如2D Moore pattern中每个数据会被访问9次。

### Transpose(转置)

Tasks re-order data elements in memory

![transpose](../../../images/2021/08/unit2-transpose.jpg)

![img](../../../images/2021/08/unit2-transpose2.jpg)

### Parallel Communitcation Patterns Recap

![summary](../../../images/2021/08/parallel_communication_patterns_recap.jpg)



## GPU Hardware

我们应该考虑什么？以下这些问题也反映出了并行程序对于性能的设计以及安全操作的要求会更高，我们要结合GPU硬件架构特性来考虑。

![](../../../images/2021/08/patterns_gpuhardware.jpg)

### Summary of programming model

线程的关键点是它们以线程块的形式出现，一个线程块是一组合作解决次要问题的线程。GPU程序会启动多个线程来运行一个kernel，然后它们都运行到完成并退出。之后程序可以启动多个线程来运行下一个kernel。

![](../../../images/2021/08/thread_blocks.jpg)

### SM

SM是GPU中很高层的核心结构，由多个SP(streaming processor, aka CUDA core，最后具体的指令和任务都是在SP上处理)加上其它一些资源组成。SM中register和shared memory是很宝贵的资源，CUDA将这些资源分配给所有驻留在SM中的线程，[这些有效的资源使得每个SM中active warps有非常严格的限制，即限制了并行能力](https://blog.csdn.net/junparadox/article/details/50540602) 。

>一个SP可以执行一个thread，但是实际上并不是所有的thread能够在同一时刻执行。Nvidia把32个threads组成一个warp，warp是调度和运行的基本单元。warp中所有threads并行的执行相同的指令。一个warp需要占用一个SM运行，多个warps需要轮流进入SM。由SM的硬件warp scheduler负责调度。目前每个warp包含32个threads（Nvidia保留修改数量的权利）。所以，一个GPU上resident thread最多只有 SM*warp个。
>————————————————
>版权声明：本文为CSDN博主「ZhangJunior」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
>原文链接：https://blog.csdn.net/junparadox/article/details/50540602

![](../../../images/2021/08/streaming_multiprocessors.jpg)

**GPU负责给SM来分配块**，所有SM都并行运行，根据定义，一个线程块只能在一个SM上运行，而一个SM可能会运行多个线程块。

CUDA对线程块将在何时、何处运行不作保证，事实上这是CUDA的一个巨大优势，是能够运行很快的原因：

| Advantages                                                   | Consequences                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| flexibility-efficiency(由于灵活性，硬件可以真正有效的运行，例如一个线程块快速完成后SM可以立即安排另一个线程块，而无需等待其他线程块完成) | 无法假设哪个block在哪个SM上运行                              |
| **scalability**: 因为无法指定线程块在哪运行或者多少线程块会同时运行 | 无法获得块之间任何明确的通信，可能会造成一些不好的后果(eg: 死锁) |

### So what does CUDA guarantee?

1. 某块中的所有线程都保证同时在同一个SM上运行。
2. 所有内核中的快都在下个内核中任何块启动前结束。



### GPU Memory Model

每个线程可以访问local memory，这是该线程专用的内存，就像其局部变量，线程可以从局部存储器读取和写入。一个线程块中的线程可以访问shared memory，它是在SM上的少量内存。任何时间整个系统中的每个线程都可以访问global memory(即通常所说的显存)。

![image](../../../images/2021/08/gpu_memory_model.png)

### Synchronization

线程需要进行同步，以避免出现后面的线程在前面的线程计算/写入结果之前就进行读取的情况。

#### Barrier

同步中最简单的一种形式。即设置一个点，程序中的线程会到这点停止并等待，直到所有线程都达到屏障点。

eg:


