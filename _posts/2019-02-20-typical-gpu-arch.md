---
layout: post
title: Modern Parallel Computing (Part 3) - Some Typical GPU Architectures
description: A series of post when I refresh my memory on the GPU. Loosely based on JDO's course material.
lang: zh
---

> 以史为鉴，可以知兴替。---李世民，《旧唐书·魏徵传》

Came out in 2010, Fermi was the first GPU architecture I learned when I started seriously doing research on parallel computing. The streaming multiprocessor design set the foundation of several GPU architectures that followed. Such as instruction cache, warp scheduler and dispatch unit, private registers, CUDA cores and SFU, shared memory and DRAM.

![fermi-arch](/public/images/fermi-arch.png){:height="50%" width="50%"}

Kepler, the successor of Fermi has proposed a more powerful next generation streaming multiprocessor (SMX) with key features such as Hyper-Q (which expands GK110 hardware work queues from 1 to 32 so that it can achieve higher utilization by being able to put different task streams on what would otherwise be an idle SMX.), dynamic parallelism (the capability for kernels to dispatch other kernels, which is vital for dynamic tasks), and NVIDIA GPUDirect.

![kepler-arch](/public/images/kepler-arch.png){:height="50%" width="50%"}

I will skip Maxwell since it is more graphics-oriented. So the next important architecture was Pascal, which came out in 2016. I think the top-3 highlight of Pascal features are: High-bandwidth Memory 2 (a stacked memory technology which enables much higher memory bandwidth), NVLink (a high-bandwidth bus between the CPU and GPU, and between multiple GPUs), and Unified memory (which enables both CPU and GPU to access a unified address without much loss of performance). Pascal is also getting used in large-scale for deep learning.

The following table shows a side-by-side comparison of several metrics among Kepler, Maxwell, and Pascal.

![kepler-maxwell-pascal](/public/images/kepler-maxwell-pascal.png){:height="50%" width="50%"}

Last but not least, Volta is the current go-to GPU for doing deep learning and other types of general-purpose computations. The biggest update from last generation was Tensor cores: units that multiplies two 4x4 FP16 matrices, and then adds a third FP16 or FP32 matrix to the result by using fused multiply-add operations, and obtains an FP32 result that could be optionally demoted to an FP16 result.

![volta-arch](/public/images/volta-arch.png){:height="50%" width="50%"}

Another important change in Volta is Independent Thread Scheduling. Pascal and earlier NVIDIA GPUs execute groups of 32 threads (known as warps) in SIMT (Single Instruction, Multiple Thread) fashion. Therefore, on Pascal and earlier GPUs, programmers need to avoid fine-grained synchronization or rely on lock-free or warp-aware algorithms.

Volta transforms this picture by enabling equal concurrency between all threads, regardless of warp. It does this by maintaining execution state per thread, including a program counter and call stack, as shown below:

![volta-its](/public/images/volta-its.png){:height="50%" width="50%"}

Volta’s independent thread scheduling allows the GPU to yield execution of any thread, either to make better use of execution resources or to allow one thread to wait for data to be produced by another. This feature opens doors to several new parallel programming patterns, which we will cover in our following blogs.

To understand more details of GPU instruction pipeline, I suggest readers to read [this awesome blog](http://taylorlloyd.ca/gpu,/pascal,/cuda/2017/01/07/gpu-pipelines.html).

I think we have pretty much finished the hardware architecture and programming model design big idea part. Next time we will start walking through CUDA and see some code that runs on the GPU. Now that we know what the GPU is, it is time to know how to efficiently write code to utilize this amazing chip!