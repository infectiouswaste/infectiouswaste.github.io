---
layout: post
title: Modern Parallel Computing (Part 2B) - How Does A GPU Work? Programming Big Ideas 
description: A series of post when I refresh my memory on the GPU. Loosely based on JDO's course material.
lang: en
---

![thread](/public/images/thread.png){:height="50%" width="50%"}

> “If you were plowing a field, which would you rather use? Two strong oxen or 1024 chickens?” ---Seymour Cray

In one sentence, the GPU programming model can be summarized as: **A massively multi-threaded processor**. So the key ideas are 1) to move data-parallel application portions to the GPU (from CPU); and 2) to organize the simultaneous execution of thousands of threads.

So how do we program on the GPU? Well, one natural thought is that we should program one thread per data element, and then write one program that runs on all threads in parallel. The software terminology for this is "SPMD": single-program, multiple-data. The hardware terminology is "SIMT": single-instruction, multiple-thread. SIMD means many threads run in lockstep; SIMT means that some divergence is allowed and handled by the hardware.

Some Definitions:
-----------------
- Device = GPU;
- Host = CPU;
- Kernel = Function that runs on the device.

A CUDA kernel is no more special than a C program for CPU, the only diffrences are that it only contains the parallel portions of an application and it is executed simultaneously by many threads on the GPU. Note that when we say thread, there are two kinds of thread: hardware thread that executes kernel on the GPU, and virtual thread that programmers define when launching a CUDA kernel. There could be millions virtual threads that defined by a programmer, but only limited number of threads that can get executed at the same time on the GPU. The most significant difference between CUDA and CPU threads is that CUDA threads are extremely lightweight. Their creation overhead is very little, and can be instantly switched to another thread. This unique characteristic forces a large number of threads for CUDA to achieve efficiency, while multi-core CPUs can use only a few.

![grid-block-thread](/public/images/grid-block-thread.png){:height="50%" width="50%"}

This illustration must be familiar to a lot of people who have been to any CUDA/GPU Programming 101 talk. At a high level, a kernel is executed as a grid of thread blocks, the programmer specifies the dimensions of the grid and thread block. A thread block is a batch of threads that can cooperate with each other through shared memory and synchronization. Two threads from two different blocks cannot cooperate because blocks get executed independently. The mapping from these CUDA software concepts to the hardware is: **Grids run on the entire machine; blocks are mapped to cores; threads are mapped to scalar processors.** We will walk through details in the following part of the post.

From the above illustration, there are two hidden goals that are not so obvious:
The first one is scalable execution. Users can write one program for any number of streaming multi-processor (SM) cores, that is to say, the program must be insensitive to the number of cores, so that it can run on any size GPU without recompiling. Here we are using a hierarchical execution model. There are three levels of hierarchies: 
- Decompose problem into sequential steps (kernels)
- Decompose kernel into computing parallel blocks
- Decompose block into computing parallel threads

Perhaps the most important thing of this execution model is that: **Hardware distributes independent blocks to SMs as available.**
So to simply put: blocks must be independent. There are two aspects of independent:
First, any possible interleaving of blocks should be valid:
- presumed to run to completion without pre-emption
- can run in any order
- can run concurrently OR sequentially 
Second, blocks may coordinate but not synchronize:
- shared queue pointer is OK. For example, eachblock can start from a different offset of a list to do computations on different data chunk.
- shared lock (A locks, B unlocks)is BAD. Since the way blocks are launched will easily cause deadlocks.

It is this independence requirement gives scalability. The goal for this design of execution model is to make the same code efficient on both hardware that runs one block at once and hardware that runs many blocks at once. 

To get a deeper understanding of this execution model, we will show the details of a SM.

![sm](/public/images/sm.png){:height="50%" width="50%"}

As we have stated before, each SM runs a block of threads on it. In modern GPUs, SMs have 8, 16, or 32 scalar processors which support IEEE 754 32-bit floating point. Currently the maximum number of threads within a SM is 1024, and all these scalar processors share a shared memory of 64KB in size. Within these threads, each group of 32 threads run at the same time (SIMD) as a warp.

There’s lots of dimensions to scale this processor with more resources. For example, we could make chips with more/less SM cores but simpler/more complicated scalar processors. We could also tradeoff between local memory size (number of registers per thread) and shared memory size.

Up until now, we have left out one critical component in the GPU: the memory spaces.
In traditional GPUs that designed for graphics, there are texture memory, constant memory, and per-thread (per-slalar processor) registers. However, for general-purpose GPUs that are designed for doing more general computations. There are two extra types of memory:
- Global Memory
- Parallel Data Cache (Shared Memory/Scratchpad)

Global memory handles fully general load/store with scatter/gather. There can be 1D, 2D, or 3D patterns for thread ID allocation. So the memory access is very flexible. These loads/scores are untyped, not limited to fixed texture types, and support pointers.

Parallel Data Cache is dedicated on-chip memory. It is explicitly managed and shared between threads for inter-thread communication. It is (much) closer to register speed than memory speed. So a CUDA ninja would always find ways to utilize this memory to speed up her CUDA programs. The shared memory has lots of benefits:
It is the living proof of using hierarchy to manage the complexity of a system. It brings the data closer to the ALU, stages computation, minimizes trips to external memory, shares values to minimize overfetch and computation, and finally, increases arithmetic intensity by keeping data close to the processors.

When thinking of memory access, one should always keep in mind the concept of memory coalescing. The mental model should be that: once you touch anything in a memory chunk, every other item in that chunk is free (or wasted depend on how you access the memory). Your memory cost is the cost of all chunks touched. The following example should provide more intuition:

![coalesce](/public/images/coalesce.png){:height="50%" width="50%"}

Suppose we want to read a 32x32 block of memory stored in row-major order, from DRAM. If we read it row-major (e.g. thread i reads the ith column), then it is fully coalesced and we only need 32 memory transactions. However, if we read it column-major (e.g. thread i reads the ith row), then it is fully uncoalesced, and we need 32x32 memory transactions. We always want coalesced read to maximize memory bandwidth utility.

We have talked about how to use shared memory to accelerate the execution of a parallel program. But to make the memory access of shared memory efficient, we have to keep in mind that shared memory on the GPU, like all other types of memory, are structured with multiple banks. When multiple threads in the same warp access the same bank, a bank conflict occurs (unless all threads of the warp access the same address within the same 32-bit word). The number of threads that access a single bank is called the *degree of the bank conflict*. Bank conflicts cause serialization of the multiple accesses to the memory bank, so that a shared memory access with a degree-n bank conflict requires n times as many cycles to process as an access with no conflict[^1].

To summarize, we have listed some facts:
- (At least for as late as Volta architecture), base unit of the hardware is a SIMD(lockstep) warp of 32 threads.
- GPU "SMs" (cores) run blocks of threads. SMs run warps when they are ready. Warps may be from the same of different blocks. The hardware schedules blocks onto SMs if there are sufficient SM resources.
- From the closest to ALUs, to the furthest, the principle of memory hierarchy is the closer it is, the smaller and faster it is, the further it is, the larger and also slower it is. The order is as follows: registers, shared memory, L1 cache, L2 cache, global memory (DRAM).

![cuda-stack](/public/images/cuda-stack.png){:height="50%" width="50%"}

The most vital ingradient of NVIDIA GPU's success is that CUDA has a really extensively rich software development kit. Nowadays, CUDA's higher level compilation phase could also be done with clang+LLVM, that gives it even more flexibility.

![cuda-compile](/public/images/cuda-compile.png){:height="50%" width="50%"}

Next time, before we really dig into CUDA and parallel algorithms, we will look back on some typical GPU designs to map our theoretical knowledge to industrial practice.

[^1]: https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_ch39.html


