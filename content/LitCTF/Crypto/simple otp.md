---
title: Simple OTP
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Crypto
draft: false
---

## Description

We all know OTP is unbreakable...

For Attachment .. It gave us a python file named `main.py` which contains the following code

```py
import random

encoded_with_xor = b'\x81Nx\x9b\xea)\xe4\x11\xc5 e\xbb\xcdR\xb7\x8f:\xf8\x8bJ\x15\x0e.n\\-/4\x91\xdcN\x8a'

random.seed(0)
key = random.randbytes(32)
```

## Solution

Basically if you use the same seed value with the same <abbr title="Random Number Generator">RNG</abbr>, you will get the same sequence of random numbers each time you run your program.

so we don't have to worry about the key value, it will the same. We just have to XOR each bytes with the corrosponding key value. This can be done by addming the following code at the bottom of the existing code.

```py
flag = bytes(a ^ b for a, b in zip(encoded_with_xor, key))
print(flag)
```

```
FLAG : LITCTF{sillyOTPlol!!!!sdfsgvkhf}
```