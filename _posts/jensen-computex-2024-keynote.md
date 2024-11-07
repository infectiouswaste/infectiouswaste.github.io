---
layout: post
title: NVIDIA at ComputeX 2024
description: Notes on Jensen Huang's ComputeX2024 Keynote
lang: en
tags: [misc]
published: false
---

GPT-4, a 2 trillion parameter model is trained with 8 trillion tokens.
Funfact, ther isn't a gigawatt datacenter on earth now, yet.
Pascal GPU will take 1000 gigawatt hours: over a month for a gigawatt DC with pascal GPUs.
Blackwell GPU uses 3 gigawatt hours to train a GPT-4 model.
10,000 GPUs, 10 days.
17,000 J for Pascal GPU per token: two light bulbs (200W) running for two days.
0.4 J per token for Blackwell GPUs.
![dgx](/assets/computex24/dgx.png)
A blackwell DGX contains 15K watts for 8 GPUs.
so each one is 1200W*8=9600W
Grace CPU will be 5400w.
MGX:
![mgx](/assets/computex24/mgx.png)
4x2 B200 GPUs * 8 = 72 B200 + NVLink Switch Gen5
![nvlink](/assets/computex24/nvlink.png)
All-Reduce on Chip
ethernet with Spectrum X:
Network level of RDMA over ethernet
Congestion control
multiple training model noise jitter reducing
![network](/assets/computex24/network.png)

Next-gen GPU will be Rubin
![hopper-blackwell-rubin](/assets/computex24/hopper-blackwell-rubin.png)


