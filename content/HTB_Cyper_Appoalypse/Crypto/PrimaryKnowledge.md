---
title: Primary Knowledge
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Crypto
draft: false
---

```
Primary Knowledge

Surrounded by an untamed forest and the serene waters of the Primus river, your sole objective is surviving for 24 hours. Yet, survival is far from guaranteed as the area is full of Rattlesnakes, Spiders and Alligators and the weather fluctuates unpredictably, shifting from scorching heat to torrential downpours with each passing hour. Threat is compounded by the existence of a virtual circle which shrinks every minute that passes. Anything caught beyond its bounds, is consumed by flames, leaving only ashes in its wake. As the time sleeps away, you need to prioritise your actions secure your surviving tools. Every decision becomes a matter of life and death. Will you focus on securing a shelter to sleep, protect yourself against the dangers of the wilderness, or seek out means of navigating the Primus’ waters?
```

The challenge code

```python
import math
from Crypto.Util.number import getPrime, bytes_to_long
from secret import FLAG

m = bytes_to_long(FLAG)

n = math.prod([getPrime(1024) for _ in range(2**0)])
e = 0x10001
c = pow(m, e, n)

with open('output.txt', 'w') as f:
    f.write(f'{n = }\n')
    f.write(f'{e = }\n')
    f.write(f'{c = }\n')
```

here we can see that the value of `n` is calculated by 

```python
math.prod([getPrime(1024) for _ in range(2**0)])
```

Simply the `range(2**0)` results in a range of 1 as `2^0 == 1`. Therefore, it generates a list with one element, which is a single 1024-bit prime number. The `math.prod` function then calculates the product of all elements in the list, but since there is only one element, it's equivalent to just taking that element itself.

As in RSA we know the value of N is calculated by the product of two prime numbers p and q. Since the value of n contains the of only one number, it is likely that the value of p and q will be the same as n, hence calculte the value of phi, which further means we can calulate the value of d or the private key.

So our 

```python
import math
from Crypto.Util.number import inverse, long_to_bytes

n = 144595784022187052238125262458232959109987136704231245881870735843030914418780422519197073054193003090872912033596512666042758783502695953159051463566278382720140120749528617388336646147072604310690631290350467553484062369903150007357049541933018919332888376075574412714397536728967816658337874664379646535347
e = 65537
c = 15114190905253542247495696649766224943647565245575793033722173362381895081574269185793855569028304967185492350704248662115269163914175084627211079781200695659317523835901228170250632843476020488370822347715086086989906717932813405479321939826364601353394090531331666739056025477042690259429336665430591623215

p = n
q = n

phi  = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)

hex_str = hex(m)[2:]  
byte_str = bytes.fromhex(hex_str)

decoded_str = byte_str.decode('utf-8')

print(decoded_str)
```

----

Source of knoledge : 

https://vm-thijs.ewi.utwente.nl/ctf/rsa

https://youtu.be/_lg2AEqRTjg