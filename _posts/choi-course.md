---
layout: post
title: Mastering SD Web UI
description: Notes when taking the course of Choi's SD Web UI course
lang: en
tags: [misc]
published: false
---



controlnet

MLSD - furniture arrangement / house repaint
Depth
![depth-cnet-example-1](../choi/cnet-depth-example-1.png)
Normal map, good for face
normal_bae: no bg, normal_midas: w/ bg
mediapipe: can be used to generate character sheet
![character-sheet](../choi/cnet-mediapipe-example-1.png)
IP2P: character with minimal bg
make it {scenes}
will create differet bgs
change haircolor
change outfit?
![ip2p](../choi/cnet-ip2p.png)
Tile resample: sementic preserving upscaler
Can be used to hide characters inside
![tile](../choi/cnet-tile.png)
reference: can be used to force image style
![ref](../choi/cnet-ref.png)
One really good looking example:
![example](../choi/cnet-t2i-example.png)
T2IA: redecoration of an indoor scene
![t2ia](../choi/cnet-t2ia.png)
inpaint: use for details, esp w/ sketch
seg: use for re-textured scenes
![seg](../choi/cnet-seg.png)
use [reference color code map](https://docs.google.com/spreadsheets/d/1se8YEtb2detS7OuPE86fXGyD269pMycAWe2mtKUj2W8/edit#gid=0) to set the scene.

Advanced Techniques:
====================

Camera:
-------

Camera view with prompt
Add "facial" in prompt to get a face only profile picture.
![shots](../choi/shots.png)
Add "cowboy hat" in negative prompt to remove cowboy hat for cowboy shot.
I2I inpaint mask: original or latent noise
extreme closeup is like your bae is just a breathe away face to face w u.
knee shot is good
use controlnet openpose to control camera view



