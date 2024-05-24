---
layout: post
title: NeRF from Core to Shell
description: Review on NeRF
lang: en
tags: [misc]
published: false
---



> I have been reading **Brain and Behavior** lately. Though I am far from reaching the part that talks about cognition mechanism in our brains, but reading this book does make me realize the probabe connections between Neural Radiance Fields (NeRF) and how our brians perceive real 3D scenes. Let me show it now.

Have you ever wondered, as we live, travel, spread our traces on this planet, how do we build mental map of a scene every time we visit a new place? Well, it is actually a complex process that involves various parts in our brain, and have taken root and sprout since we were little. The term, I believe, is called _spatial cognition_. Let's try to break down what happens to our brain while we touring a new place:
 - Our eyes perceive a stream of image pairs that contain RGB color values and create depth maps of objects in front of us;
 - A part in our brain keeps track of the direction in which we are facing. Or to put it this way: it correctly places our eyes in the world coordination;
 - By keep integrating these incoming information, other parts in our brain gradually build and optimize an encoded visual map of the scene.

The above breakdown is, of course, a simplified version that ignores how exactly the visual map is generated, but it does highlight the input and output of our spatial cognition system: using a series of spatially aligned images (and optionally depth maps), our brain could generate and update a visual map, or shall we say, a neural representation of the 3D scene we are exploring. If you have read literature on NeRF, it does exactly the same thing! In the context of computer graphics and AI, NeRF synthesizes 3D scenes from camera-calibrated 2D images using neural networks. In the context of neural science, NeRF compresses a visual scene perceived by a series of spatially aligned images into a neural network, and imagine in our head how the scene would be like when viewed from a different angle!

### The History

> Plus ça change, plus c'est la même chose. -Alphonse Karr

It is normal for $N$-dimensional living creatures to perceive their surroundings with at most $N-1$ dimension information. This is a consequence of the spatial limitations imposed by the dimensionality of the universe we inhabit. Since the origin of our civilization, our brains have served as the only means to understand $N$-dimension information through the input of $N-1$ dimension information, until there appeared _Computer Graphics_.

Computer Graphics has a long and rich history, with an immense body of research literature and textbooks. It involves simulating the interaction of light with surfaces, materials, and cameras, using mathematical models and algorithms, to produce photorealistic or stylized images. So in essence, computer graphics seeks to mimic natural phenomena by simulating the propagation of light (photons) through a scene, its interaction with various objects, and quantifying the resulting color and intensity of light perceived at any given position of observation. This process is called _rendering_.

There are two things that worth noting here:

 - The input and the output of a rendering system is reversed from our visual cognition system. Hence the newly emerged field called _inverse rendering_, which tries to go from the output of a rendering system: images to the input of the system: scene representation, by learning the rendering process.
 - Although James Kajiya introduced the famous rendering equation back in 1986, which can very accurately describe the rendering process, in reality, solving this integral equation is notoriously difficult due to its high-dimensional and recursive nature.

 $$
 L_o(\mathbf{x}, \mathbf{\omega_o}) = L_e(\mathbf{x}, \mathbf{\omega_o}) + \int_{\Omega} f_r(\mathbf{x}, \mathbf{\omega_o}, \mathbf{\omega_i}) L_i(\mathbf{x}, \mathbf{\omega_i}) (\mathbf{\omega_i} \cdot \mathbf{n}) \, d\mathbf{\omega_i}
 $$

Therefore, it's clear that to learn the rendering process, we need to define it in a way that allows for iterative optimization according to an objective function, and that can converge in a reasonably limited amount of time, just like we do for every other machine learning system. A high-demensional recursive rendering equation, despite its elegance, can never provide such a solution.

Compared to his 1986 seminal paper [The Rendering Equation](https://doi.org/10.1145/15886.15902), Kajiya also has a paper on volume rendering equation in 1984: [Ray Tracing Volume Densities](https://doi.org/10.1145/964965.808594). Its section 3.2 has almost the same volume rendering equation in the NeRF paper. This was 28 years before NeRF was invented! They even share the same assumptions: 1) uniform medium; 2) handles only absorsion and emission; 3) uses quadrature to compute the integral. In 2022, [Ben](https://bmild.github.io/) and [Andrea](https://taiya.github.io/) published a beautiful [two-pager](https://arxiv.org/pdf/2209.02417.pdf) on how to derive the integral equation in NeRF paper from the probablistic and volume rendering perspectives.

Recall the equation, if simplifed with the above-mentioned assumptions, can be written as follows:

[//]: # (put NeRF's equation 4 and equation 5 here)

### The Core

[//]: # (NeRF pipeline in details)

### The Shell

[//]: # (NeRF application, may not include too many details)

### The Gap

[//]: # (NeRF in 5 years, when to not use NeRF, the fundamental 3D model)

### The End




