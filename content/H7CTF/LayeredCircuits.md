---
title: Layered Circuits
date: 2024-09-27
author: 1byteBoy
tags:
  - Hardware
draft: false
---

## Description

Layers conceal the flag you seek, decode the PCB to speak.

## Intuition

The challenge gave us a zip file containing the following files

```
├── ctf gerber
│   ├── CTF LED matrix-B_Adhesive.gbr
│   ├── CTF LED matrix-B_Cu.gbr
│   ├── CTF LED matrix-B_Mask.gbr
│   ├── CTF LED matrix-B_Paste.gbr
│   ├── CTF LED matrix-B_Silkscreen.gbr
│   ├── CTF LED matrix-Edge_Cuts.gbr
│   ├── CTF LED matrix-F_Adhesive.gbr
│   ├── CTF LED matrix-F_Cu.gbr
│   ├── CTF LED matrix-F_Mask.gbr
│   ├── CTF LED matrix-F_Paste.gbr
│   ├── CTF LED matrix-F_Silkscreen.gbr
│   ├── CTF LED matrix-In1_Cu.gbr
│   ├── CTF LED matrix-In2_Cu.gbr
│   ├── CTF LED matrix-NPTH-drl.gbr
│   ├── CTF LED matrix-NPTH-drl_map.gbr
│   ├── CTF LED matrix-PTH-drl.gbr
│   ├── CTF LED matrix-PTH-drl_map.gbr
│   ├── CTF LED matrix-drl.rpt
│   └── CTF LED matrix-job.gbrjob
└── layers.jpg
```

![layers.jpg](/images/h7ctf/layers.jpg)

## Solution

First thing is to search for, what gbr file extension stands for and what is the usecase of files with that extension.

`A file with . gbr extension is a Gerber image file format for exchange of printed circuit board (PCB) design data transfer`

Next i searched, how to open gbr files and found a great Free and Open Source software named [gerbv](https://gerbv.github.io/).

With that software installed, i opened all the gbr files at once, cause that's how it should be done, this is because each of those files is a layer for the PCB design. 

We can see that some of those layers contains a base64 encoded string, mainly those that are mentioned in the image we got. After decoding each base64 encoded string and piecing it togther we got the flag

`H7CTF{h4rdw4r3_ch4lls_4r3_1nd33d_4_th1ng}`

