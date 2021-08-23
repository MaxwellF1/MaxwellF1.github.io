# Udacity CS344-Intro to Parallel Programming-Unit1-The GPU Programming Model


受功耗限制，不能无限提高单块芯片的频率，不如用很多处理器来进行计算。

> Seymour Cray: Would you rather plow a field with two strong oxen or 1024 chickens?

## What kind of Processors are we Building

Major design constraint: Power.

不采用CPU是因为CPU有复杂的控制单元硬件，这允许性能的灵活性，但随着控制硬件变得复杂，耗电量也会增高。不如采用更简单的结构，并将这些重复的简单结构放在一个芯片中，但这需要新的编程模型。

需要优化什么？

Latency(time, seconds): 即最大程度缩小执行时间。传统的CPU主要是在这一点，试图最小化每个任务具体消耗的时间。

Throughout(stuff/time, jobs/hour): 每个单位时间完成的工作量。GPU主要来优化吞吐量。

GPU的设计特点：

1. GPU有很多简单计算单位，可以在一起执行大量的计算(GPU一般愿意来用更简单的控制硬件来换取更多的计算单元，这也对程序员的要求更高)。

2. GPU有显式编程模型，在给机器编写程序时，我们知道我们有很多处理器，并且根据这一点进行编程。

3. GPU为吞吐量进行优化，而不是延迟(它愿意接受任何单一个体计算增加的延迟来换取单位时间内进行更多的计算)。

   

## CUDA

如果还是普通的c语言编程，只能在CPU("host")上运行，而CUDA帮助我们在GPU("device")上进行编程。

![image-20210821230341935](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210821230341935.png)

此图大概说明了CUDA的整体工作结构。注意，GPU只能响应CPU发送数据或接收数据的请求而不能initiate这些请求。

### A CUDA Program

![image-20210821231005887](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210821231005887.png)

注意图中的通信很可能是有很大开销的，所以要注意编程中的计算和通信的比例。

![image-20210821231535299](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210821231535299.png)

### What is the GPU good at?

1. 有效的启动大量线程("The GPU does not even get out of bed in the morning for fewer than a thousand threads").
2. 同时并行运行大量线程。

## GPU Calculation

### GPU Code: A High-level View

![image-20210822121600138](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210822121600138.png)

我们启动的每一个线程知道自己的编号，即有线程索引。大概流程是先编写在一个线程上运行的kernel，然后会启动许多线程，每个线程独立运行那个内核。

### Squaring Numbers Using CUDA

```c
#include <stdio.h>
 
__global__ void square(float* d_out, float* d_in){
  int idx = threadIdx.x;
  float f = d_in[idx];
  d_out[idx] = f * f;
}
 
int main(int argc, char** argv){
  const int ARRAY_SIZE = 64;
  const int ARRAY_BYTES = ARRAY_SIZE * sizeof(float);
 
  // generate the input array on the host
  float h_in[ARRAY_SIZE];
  for(int i = 0; i<ARRAY_SIZE; i++){
    h_in[i] = float(i);
  }
  float h_out[ARRAY_SIZE];
 
  // declare GPU memory pointers
  float* d_in;
  float* d_out;
 
  // allocate GPU memory
  cudaMalloc((void **) &d_in, ARRAY_BYTES);
  cudaMalloc((void **) &d_out, ARRAY_BYTES);
 
  // transfer the array to GPU
  cudaMemcpy(d_in, h_in, ARRAY_BYTES, cudaMemcpyHostToDevice);
 
  // launch the kernel
  square<<<1, ARRAY_SIZE>>>(d_out, d_in);
 
  // copy back the result array to the GPU
  cudaMemcpy(h_out, d_out, ARRAY_BYTES, cudaMemcpyDeviceToHost);
 
  // print out the resulting array
  for(int i=0; i<ARRAY_SIZE; i++){
    printf("%f", h_out[i]);
    printf(((i % 4) != 3) ? "\t" : "\n");
  }
 
  // free GPU memory allocation
  cudaFree(d_in);
  cudaFree(d_out);
 
  return 0;
 
 
}
```

这是一个计算平方的程序，但可以大概看出CUDA程序的结构。

#### Configuring the kernel call

square.cu中我们启动了一个块，这个块含有64个线程。但参数是我们可以自行调整的。

需要注意，可以同时运行很多块，而且每个块是有支持的线程上限的，例如512,1024。

```c
Kernel<<<Grid of Blocks, Block of Threads>>>(...)
```

CUDA不仅支持1维的线程块，还支持2、3维的线程块，同样的我们也可以把线程块安排到1、2、3维的网格中(当我们不指定时，默认为1维)。我们需要三维结构时，可以通过如下方式借助dim3格式来初始化(我们不指定时则默认y,z为1)。

```c
dim3(x,y,z)
dim3(w,1,1) == dim3(w) == w
square<<<1,64>>> == square<<<dim3(1,1,1), dim3(64,1,1)>>>
```

所以通用的内核启动可以看成如下三个参数的形式：

```c
square<<<dim3(bx,by,bz), dim3(tx,ty,tz), shmem>>>(...)
```

其中第一个参数是网格块的维数(即网格中有bx×by×bz个线程块)，每个线程块的维数由第二个参数指定，第三个参数是以字节为单位的给每个线程块分配的共享内存量(默认为0)。当我们的问题有多个维度时，拥有多维网格和块是很方便的。

![image-20210822150630400](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210822150630400.png)

## Map

![image-20210822151938948](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210822151938948.png)

## Summary

![image-20210822152210683](C:\Users\Maxwell\AppData\Roaming\Typora\typora-user-images\image-20210822152210683.png)

## Problem set 1

将彩色照片变为黑白的，这次作业比较简单。但自己花费了很多时间在环境搭建上，没想到Windows居然搞好了，但是不想用过于庞大繁琐的Visual Studio，就花了很多时间在WSL2+CUDA的配置上，但现在还是没有完全配置好环境，只能继续用IDE了。此外，某位大佬用Colab好像配了个jupeternotebook那样的环境，只要根据报错将makefile里面的compute_30改为compute_35就可以正常使用了。

下面是需要编写的student.cu的代码：

```c

// Homework 1
// Color to Greyscale Conversion

//A common way to represent color images is known as RGBA - the color
//is specified by how much Red, Grean and Blue is in it.
//The 'A' stands for Alpha and is used for transparency, it will be
//ignored in this homework.

//Each channel Red, Blue, Green and Alpha is represented by one byte.
//Since we are using one byte for each color there are 256 different
//possible values for each color.  This means we use 4 bytes per pixel.

//Greyscale images are represented by a single intensity value per pixel
//which is one byte in size.

//To convert an image from color to grayscale one simple method is to
//set the intensity to the average of the RGB channels.  But we will
//use a more sophisticated method that takes into account how the eye
//perceives color and weights the channels unequally.

//The eye responds most strongly to green followed by red and then blue.
//The NTSC (National Television System Committee) recommends the following
//formula for color to greyscale conversion:

//I = .299f * R + .587f * G + .114f * B

//Notice the trailing f's on the numbers which indicate that they are
//single precision floating point constants and not double precision
//constants.

//You should fill in the kernel as well as set the block and grid sizes
//so that the entire image is processed.

#include "utils.h"

__global__
void rgba_to_greyscale(const uchar4* const rgbaImage,
                       unsigned char* const greyImage,
                       int numRows, int numCols)
{
  //TODO
  //Fill in the kernel to convert from color to greyscale
  //the mapping from components of a uchar4 to RGBA is:
  // .x -> R ; .y -> G ; .z -> B ; .w -> A
  //
  //首先找到本线程对应需要处理的像素
  int x = threadIdx.x + blockDim.x*blockIdx.x;
  int y = threadIdx.y + blockDim.y*blockIdx.y;
  int Idx = x + numCols*y;
  uchar4 rgba = rgbaImage[Idx];
  //The output (greyImage) at each pixel should be the result of
  //applying the formula: output = .299f * R + .587f * G + .114f * B;
  //Note: We will be ignoring the alpha channel for this conversion
  greyImage[Idx] = 0.299f*rgba.x + 0.587f*rgba.y + 0.114f * rgba.z;
  //First create a mapping from the 2D block and grid locations
  //to an absolute 2D location in the image, then use that to
  //calculate a 1D offset
}

void your_rgba_to_greyscale(const uchar4 * const h_rgbaImage, uchar4 * const d_rgbaImage,
                            unsigned char* const d_greyImage, size_t numRows, size_t numCols)
{
  //You must fill in the correct sizes for the blockSize and gridSize
  //currently only one block with one thread is being launched
  const dim3 blockSize(16, 16, 1);  //TODO
  const dim3 gridSize( floor(numCols/16)+1,floor(numRows/16)+1, 1);  //TODO
  rgba_to_greyscale<<<gridSize, blockSize>>>(d_rgbaImage, d_greyImage, numRows, numCols);

  cudaDeviceSynchronize(); checkCudaErrors(cudaGetLastError());
}
```




