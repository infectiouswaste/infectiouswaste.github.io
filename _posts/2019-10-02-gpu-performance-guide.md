---
layout: post
title: GPU Programming - Performance Guide
description: This article is heavily inspired by the same titled article by Sean Baxter in moderngpu 1.0 wiki (https://moderngpu.github.io/performance.html)
lang: en
---

CPUs are latency-oriented systems. Hence performance optimization on CPUs uses large caches, branch prediction, and pre-fetching to avoid stalls on data dependencies. GPUs, by contrast, are throughput-oriented systems that use massive parallelism to hide latency. When doing performance optimization on GPUs, there are two levels of parallelism:

- Thread-level Parallelism: the number of threads run in parallel on a GPU. This is also known as __Occupancy__.
- Instruction-level Parallelism: the measure of workload inside one thread.

An ideal task for running on GPUs should have both high occupancy (can be decomposed into many independent sub-tasks) and high ILP (each sub-task has enough workload to maximize ILP). To put in simple words, we want all threads to be fully occupied and to have more compute and load/store in each cycle.

There are several limiting factors for GPU kernel performance, to name a few:
- DRAM to L2 cache bandwidth;
- L2 to SM bandwidth;
- texture bandwidth;
- FFMA/DFMA throughput;
- Integer throughput;
- shared memory performance.


First we analyse the resource limits that cap occupancy:

|                             | sm_20 (Fermi) |  sm_35 (Kepler) | sm_61 (Pascal) | sm_75 (Volta) | 
|-----------------------------|---------------|-----------------|----------------|---------------|
| Max Threads (SM)            | 1536          | 2048            | 2048           | 1024          |
| Max CTAs (SM)               | 8             | 16              | 32             | 16            |
| Shared Memory Capacity (SM) | 48K           | 48K             | 96K            | 64K           |
| Register File Capacity (SM) | 32K           | 64K             | 64K            | 64K           |
| Max Registers (Thread)      | 63            | 255             | 255            | 255           |

- __Max Threads__ Even using max threads per SM, GPUs can still be under-occupied due to poor ILP. We can increase the program's parallelism by __register blocking__ (to process multiple elements per thread). We can also reduce the CTA size, so that barriers do not stall as many threads: smaller CTAs lead to better overlapped execution than larger ones.

- __Max CTAs__ When number of threads per block is smaller than (Max Threads / Max CTAs), it reduces occupancy; meanwhile larger blocks compromise overlapped execution. 128 or 256 threads per block is a good number to start with.

- __Shared Memory Capacity__ GPU SMs usually increase their arithmetic throughput and latency together, but if shared memory capacity does not increase alongside, it will affect the actual max number of threads on SM. For example, using Kepler architecture, at 100% occupancy, a thread has 32 registers (128 bytes) but only 24 bytes of shared memory. If we do register blocking for eight values per thread, then there can be no higher than 1536 threads/SM occupancy (75%) or we will surpass our shmem capacity. More aggressive register blocking drops this further. This is a good example of the tradoff between ILP and occupancy.

- __Register File Capacity__ The design of GPU follows the inverted cache hierarchy, meaning that the cache that closer to computing cores are more copious. Register file capacity is larger than even L2 cache. Still, users should carefully write kernels to reduce register usage. One way to optimize this is re-ordering load and store procedures to move out results before reading more inputs.

- __Max Registers__ Kernels that are limited by RF capacity usually cannot reach maximum occupancy (launch max threads on a SM), for GPUs with compute capability >= 3.0, due to a larger number of max registers per thread, achieved occupancy may get even lower, because each thread could consume four times as much of the RF than previous generation GPUs.

CTA size and shared memory consumption are specified by the programmer; these are easily adjusted. RF usage is an implicit parameter. The `__launch_bounds__` kernel attribute gives the user more control over occupancy by providing a cap on per-thread register usage. Tag the kernel with the CTA size and the desired number of CTAs per SM. The code generator now caps register usage by re-ordering instructions to reduce live variables. It spills the overflow.

Second, we analyse factors that influence ILP:

Here we keep several parameters from moderngpu library: NT (number of threads per CTA), VT (values per thread, the grain size), and NV( NT*VT, number of values per CTA), is the tile size. Increasing tile size amortizes the cost of once-per-thread and once-per-CTA operations, improving work efficiency. On the other hand, increasing grain size consumes more shared memory and registers, reducing occupancy and the ability to hide latency.

For each kernel, there are three tuning parameters: NT, VT, and `__launch_bounds__` argument OCC (minimum number of CTAs per SM). Traditionally, finding optimal tuning parameters is an empirical process. Since different hardware architectures, data types, input sizes and distributions, and compiler versions all affect parameter selection. Could AutoML help with this search process?

In our next post, we will focus on one fundamental design methodology for GPU programming: two-phase strategy. Which separates the scheduling/partitioning from the actual problem-solving logic. After that, we will start talking about several common primitives in GPU computing.
