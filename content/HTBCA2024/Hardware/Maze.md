---
title: Maze 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Hardware
draft: false
---

```
Maze

In a world divided by factions, "AM," a young hacker from the Phreaks, found himself falling in love with "echo," a talented security researcher from the Revivalists. Despite the different backgrounds, you share a common goal: dismantling The Fray. You still remember the first interaction where you both independently hacked into The Fray's systems and stumbled upon the same vulnerability in a printer. Leaving behind your hacker handles, "AM" and "echo," you connected through IRC channels and began plotting your rebellion together. Now, it's finally time to analyze the printer's filesystem. What can you find?
```

Download and extract the file. 

Run `tree` command. 

```
└── fs
    ├── PJL
    ├── PostScript
    ├── saveDevice
    │   └── SavedJobs
    │       ├── InProgress
    │       │   └── Factory.pdf
    │       └── KeepJob
    └── webServer
        ├── default
        │   └── csconfig
        ├── home
        │   ├── device.html
        │   └── hostmanifest
        ├── lib
        │   ├── keys
        │   └── security
        ├── objects
        └── permanent
```

we can spot the `Factory.pdf` file, on opening and reading through the file, we get the flag at on of the page.