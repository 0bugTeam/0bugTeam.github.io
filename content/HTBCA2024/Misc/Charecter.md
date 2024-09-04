---
title: Charecter 
date: 2024-03-14
author: 1byteBoy
tags:
  - Misc
draft: false
---

## Description

Character

Security through Induced Boredom is a personal favourite approach of mine. Not as exciting as something like The Fray, but I love making it as tedious as possible to see my secrets, so you can only get one character at a time!

## Solution

It gives us and IP and a Port through which we can connect using netcat.

It gives us the following choice and on answering it gives each letter of the flag.

```
Which character (index) of the flag do you want? Enter an index: 0
Character at Index 0: H
.....
```

Now this is very tidious job as the flag will probably be very long, so we can write a script for this

```python
from pwn import *

target = remote("94.237.48.205", 39652)

flag       = []
temChar    = []
index      = 0

while True:
    target.sendline(str(index))
    char = target.recv(1).decode('utf-8')

    if char == '}': 
        flag.append(char)
        break
    if char == '\n' :
        flag.append(temChar[-1])
        temChar.clear()

    temChar.append(char)
    index += 1

print(''.join(flag))
target.close()
```

Which will print the flag in proper format..