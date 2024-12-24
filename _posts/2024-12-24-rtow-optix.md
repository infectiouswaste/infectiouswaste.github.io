---
layout: post
title: RayTracing on a Weekend - OptiX 8
description: An introduction for using OptiX 8 to build a raytracer for human
lang: en
tags: [tutorials]
published: false
---


<style>
.highlight-left {margin-left: 0}
</style>

> I am not an artist, but I can paint with algorithms.  ---Liyi Wei

As John Lasseter has famously said: The art challenges the technology, and the technology inspires the art. In my opinion, there is no better thing on this planet than a ray tracer (or a path tracer) that reflects the quote more vividly. From the science perspective, ray tracing algorithm tries to simulate the behavior of the light using physically-based methods; from the engineering perspective, ray tracing is nicely object oriented with natural object-like components such as rays, shapes, materials, colors, motions, and cameras.

From an educational standpoint, this field sets a relatively high bar for beginners. You need to be comfortable with calculus, physics, and statistics to understand the underlying theory, and you also need some familiarity with C++ and CUDA to work with mainstream ray tracer codebases. On top of that, a basic grasp of GPU and parallel computing is important if you don’t want to wait all night for a single 512×512 image of glass spheres to finish rendering. Fortunately, there are several excellent books and tutorials available. Two standout examples include [Physically Based Rendering](https://pbr-book.org/) and [Ray Tracing in One Weekend](https://raytracing.github.io/), each providing invaluable guidance on essential concepts. However, there’s still a gap between the coverage in these resources and real-world industrial practices. NVIDIA’s OptiX library is the state-of-the-art solution in ray tracing, offering users full control over GPU-based ray tracing algorithms, but—like CUDA—it is not particularly beginner-friendly. This is where owl, created by the great Ingo Wald, steps in. Owl provides a more approachable path for newcomers, inspired several projects that followed, and even includes examples taken from Ray Tracing in One Weekend. Yet, these examples don’t always draw a clear line between important concepts and the working C++ code. In response, this tutorial series aims to bridge that gap, giving beginners a detailed explanation of how to build an efficient ray tracer with OptiX using owl as a foundation.

Before We Start
---------------

First make sure you have CUDA (preferrably 12.4+) and OptiX 8 installed on your computer and correctly set the library paths.

```
export PATH=/usr/local/cuda-12.4/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:$LD_LIBRARY_PATH
export OptiX_INSTALL_DIR=/path/to/optix8
```





