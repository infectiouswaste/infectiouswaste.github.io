---
layout: post
title: Modern Parallel Computing (Part 1) - A Bit of History
description: A series post when I refresh my memory on GPU. Loosely based on JDO's course material.
lang: en
---

> And since geometry is the right foundation of all painting, I have decided to teach its rudiments and principles to all youngsters eager for art. ---Albrecht Dürer

![early-graphics-hw](/public/images/early-graphics-hw.jpg){:height="50%" width="50%"}
*Artist using a perspective machine. Albrecht Dürer, 1525*

Since Darwin's age, people have been using graphics hardware for creating pictures. Early such grahics hardware includes perspective machine. If you look closely enough, it has several features of modern graphics hardware: it does the transformation of perspective, and it is discrete (e.g. sampling on the surface of the objects one wants to draw, then finishing the whole drawing by smoothing).

![early-electronic-graphics-hw](/public/images/sketchpad.png){:height="50%" width="50%"}
*Ivan Sutherland with his SKETCHPAD. 1963*

Ivan Sutherland, the creator of modern computer grapchis, has developed SKETCHPAD, an early electronic graphics hardware for human-computer interaction as well as CAD. While it was basically in 2D, people have soon started 3D rendering and created "graphics pipeline".

![geometry-engine](/public/images/geometry-engine.png){:height="50%" width="50%"}
*Geometry Engine and graphics pipeline it uses. Jim Clark, 1982*

SGI has developed huge graphics workstation which contains the graphics hardware shown above: Geometry system. It is one of the earliest graphics pipeline. Each block is one geometry engine, and the whole pipeline starts with some geometry transformation (operation done per vertex in the matrix form), followed by rasterization (operation done on each pixel) and clipper (occlusion/depth test etc.). For a long time, graphics pipeline has been fixed in that form.

![graphics-pipeline](/public/images/pipeline.png){:height="50%" width="50%"}
*A Typical fixed graphics pipeline.*

Going through any book on graphics rendering. You would probably encounter an illustration more or less similar to this one. When I started learning graphics, I first came across this too. Basically this is still the expansion of geometry engine, where you start by some vertex-related gemoetry transformation/lighting, then perform triangle setting and rasterization, then texture/pixel shading, then depth test and alpha blending, before you finally send RGB values for each pixel to the framebuffer for rendering. Here I will omit some famous graphics system during this era such as RealityEngine Graphics and InfiniteReality.

![programmable-pipeline](/public/images/programmable-pipeline.png){:height="50%" width="50%"}
*A Typical programmable graphics pipeline.*

Soon enough people are not satisfied with fixed data going through this pipeline: they want to do arbitrary per-vertex and per-pixel operations, this is the beginning of programmable graphics pipeline, and these programs are called: shaders. There are some extra stages too, such as tessellation stage, where you can change the granularity of polygons you want to render.

![unified-shading](/public/images/unified-shading.png){:height="50%" width="50%"}

With all these improvements along the way, people start to realize that **GPU architecture increasingly centers around shader execution**. Instead of doing all these shading passes discretely, can we unify the runnings with a general form of execution?
In graphics, this is called unified shader and outside graphics, people already started exploring a cool new stuff called GPGPU (general purpose graphics processing unit), which will light the fire of modern parallel computing and help boost a new wave of computing revolution: deep learning.

Next time we will really jump into the rabbit hole of modern parallel computing and go through some big ideas behind the design of massively parallel chips such as NVIDIA's GeForce GPU and Intel's Larrabee and Xeon Phi.

