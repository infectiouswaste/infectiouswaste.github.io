---
layout: post
title: Modern Parallel Computing (Part 2) - How Does A GPU Work?
description: A series of post when I refresh my memory on GPU. Loosely based on JDO's course material.
lang: en
---

> “If you were plowing a field, which would you rather use? Two strong oxen or 1024 chickens?” ---Seymour Cray

The challenge of modern GPU design is not how to pack enough arithmetic on a chip but instead how to do so in an energy-efficient way since the communication is the dominant power cost. For this reason, one must exploit locality. Let's put it this way: the other side of chip is many cycles away, and off-chip is hundreds of cycles away. Looking back from the first day that CUDA and general purpose GPU computing was born till now, the big iead has always been: **GPUs are designed to maximize throughput as opposed to minimize latency.** In the remaining part of this post, we will take a tour to go over some of the big ideas in GPU's hardware design.

Before that, let's first take a look at a CPU-style chip. We see module that fetch and decode instruction, ALU for arithmetic and logic computation, execution context to store context for different processes. And a bunch of components such as OOO, branch predictor, pre-fetcher, etc. to make a single instruction stream run fast.

![cpu-style-chip](/public/images/cpu-style-chip.png){:height="50%" width="50%"}

The first hardware big idea is to **remove components that help a single instruction stream run fast.** Why doing so? Because we are trading off that space on the chip with many, many essential components, by essential, I mean fetch/decode, ALU, and execution context.

![multi-thread](/public/images/multi-thread.png){:height="50%" width="50%"}

As the number of threads gets larger, one obvious observation is that, threads that run together should be able to share a common instruction stream. This is not new for GPU and is called [SIMD](https://en.wikipedia.org/wiki/SIMD) (Single Instruction, Multiple Data). Doing so can amortize cost/complexity of managing an instruction stream across many ALUs. We would call the design in the following picture “8-wide” SIMD, while NVIDIA would say this is a “warp” of size 8.

![simd](/public/images/simd.png){:height="50%" width="50%"}

So the second hardware big idea is to **use SIMD to amortize cost of managing an instruction stream across many ALUs.** Now we have two levels of parallelism:
  - SIMD within a core
  - Multiple cores

If we build a chip using our current two big ideas, it would look like this:

![two-level-parallelism](/public/images/two-level-parallelism.png){:height="50%" width="50%"}

This chip has:
- 16 cores
- 8 MAD ALUs/core
- 16 simultaneous instruction streams
- 64 (16*4) “resident” (but interleaved) instruction streams
- 512 (64*8) resident threads
- 256 GFLOPS @ 1 GHz (without interleaved streams, 16 * 8 * 2 FLOPs/cycle * 1G clock rate)


