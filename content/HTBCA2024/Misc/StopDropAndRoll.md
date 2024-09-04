---
title: Stop Drop and Roll 
date: 2024-03-14
author: 1byteBoy
tags:
  - Misc
draft: false
---

## Description

Stop Drop and Roll

The Fray: The Video Game is one of the greatest hits of the last... well, we don't remember quite how long. Our "computers" these days can't run much more than that, and it has a tendency to get repetitive...

## Solution

Connectig to the netcat instance show the following

```
===== THE FRAY: THE VIDEO GAME =====
Welcome!
This video game is very simple
You are a competitor in The Fray, running the GAUNTLET
I will give you one of three scenarios: GORGE, PHREAK or FIRE
You have to tell me if I need to STOP, DROP or ROLL
If I tell you there's a GORGE, you send back STOP
If I tell you there's a PHREAK, you send back DROP
If I tell you there's a FIRE, you send back ROLL
Sometimes, I will send back more than one! Like this: 
GORGE, FIRE, PHREAK
In this case, you need to send back STOP-ROLL-DROP!
Are you ready? (y/n) y
Ok then! Let's go!
FIRE, GORGE, PHREAK, PHREAK, PHREAK
What do you do?
```

Now the thing the challenge is asking us to do is very challenging to do manually. so we can write the following script to automate the whole thing

```python
from pwn import *

mapping = {'GORGE': 'STOP', 'PHREAK': 'DROP', 'FIRE': 'ROLL'}

target = remote("94.237.57.59", 58302)

target.recvuntil("Are you ready? (y/n) ".encode('utf-8'))
target.sendline('y'.encode('utf-8'))

# Skip the first line of the game
_skip_ = target.recvline().decode('utf-8')

while True:
    # Receive the scenarios and print them
    scenarios = target.recvline().decode('utf-8').strip()
    if 'HTB' in scenarios : 
    	print(scenarios)
    	break
    scenarios = scenarios.split(',')
    
    # Map each scenario to its corresponding response
    responses = '-'.join(mapping[scenario.strip()] for scenario in scenarios)

    # Print the responses
    target.recvuntil("What do you do? ".encode('utf-8'))
    print("Responses:", responses)
 
    # Send the responses back to the server
    target.sendline(responses.encode('utf-8'))

target.close()
```

The script will print flag at the end!