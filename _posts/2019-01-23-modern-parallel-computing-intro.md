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



