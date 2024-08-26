---
title: "Anarkhia"
date: 2021-12-11
draft: false
tags: ["Cryptography"]
categories: ["Cryptography"]
summary: "Solved Anarkhia in Oddessy CTF"
author: "1byteBoy"
---

## Challenge Description

Amidst the swirling tempest of chaos, seeds of creation take root, birthing new possibilities from the depths of uncertainty.

## Solution

We were provided with zip file [Anarkhia.zip](https://odyssey.hackrocks.com/media/challenges/CTF-ODYSSEY-04/content/Anarkhia.zip), which contains two files one being `out.txt` which contains the following data

```
0x5e0x1d40x1940xcc0xca0x13a0x1340xea0xd40x19a0x1220x1560x1c40xca0x880x9c0x1680x1220x220x1d20xd00x1b00x1c00xd20xb80x1a0xea0x1060x1080xd20x7e0x1ce0x920x1b00x8a0x760x1ac0x1420x1fe0xd20x12a0x14c0x340x4c0x1d00x1060x440x340x7a0x1b0
```

and other is the `enc.py` which has the following code

```python
#!/usr/bin/python3

import random
from secret import flag

random.seed(''.join([str(random.randint(0x0, 0x9)) for i in range(random.randint(3, 6))]));theKey = [random.randint(0, 255) for i in range(len(flag))];theEnc = "".join([hex(((random.choice(theKey)) ^ ord(flag[i]))<<1) for i in range(len(flag))]);open('out.txt', 'w').write(theEnc)
```

So basically <u>out.txt</u> contains hex string which is the flag encrypted with XOR technique, but shifting and the key being a random number.

we can split the code to understand it more properly

```python
#!/usr/bin/python3

import random
from secret import flag

# Gererate the random seed
seed_value = ''.join([str(random.randint(0x0, 0x9)) for i in range(random.randint(3, 6))])

# Set the seed for random generation
random.seed(seed_value)

# Generate the random array of key for encryption
theKey = [random.randint(0, 255) for i in range(len(flag))]

# Encrypt the flag
theEnc = ""
for i in range(len(flag)):

	# Select a random key value from theKey array
	random_key_value = random.choice(theKey)

	# Get the Unicode code point of the current character of flag
	flag_char_code = ord(flag[i])

	# XOR the random key value with the Unicode code point
	xor_result = random_key_value ^ flag_char_code

	# Left shift the XOR result by one bit 
	shifted_result = xor_result << 1

	# Convert the shifted result to its hexadecimal representation
	hex_representation = hex(shifted_result)

	# Append the hexadecimal representation to theEnc
	theEnc += hex_representation

# Write the encrypted flag to a file named "out.txt"
with open('out.txt', 'w') as file:
file.write(theEnc)
```

After taking a lot of time to understand the code i wrote a code to reverse the process and get the flag

```python
import re
import random

# Encrypted flag in hex value
enc_flag = "0x5e0x1d40x1940xcc0xca0x13a0x1340xea0xd40x19a0x1220x1560x1c40xca0x880x9c0x1680x1220x220x1d20xd00x1b00x1c00xd20xb80x1a0xea0x1060x1080xd20x7e0x1ce0x920x1b00x8a0x760x1ac0x1420x1fe0xd20x12a0x14c0x340x4c0x1d00x1060x440x340x7a0x1b0"

# Function to convert the hex string to a list of integers
def hex_string_to_int_list(hex_string):
	hex_values = hex_string.split("0x")[1:]
	return [int(hex_val, 16) for hex_val in hex_values]

# Convert the hex string to a list of integers
integer_flag_values = hex_string_to_int_list(enc_flag)

# Right Shifting the integer values
rs_integer_flag_values = [val >> 1 for val in integer_flag_values]

length_of_flag = len(integer_flag_values)

dec_flag = ''

init_count = 1000
padd_val = 3
seed_val = 0

# note : the values can also have 0's before them like 001028
while True:

    if seed_val == init_count:
        seed_val = 1
        init_count = init_count * 10
        padd_val = padd_val + 1

    print(f"Trying with seedvalue: {seed_val:0{padd_val}d}")
    random.seed(f"{seed_val:0{padd_val}d}")
    theKey = [random.randint(0, 255) for i in range(length_of_flag)]

    for i in range(length_of_flag):
        random_key_value = random.choice(theKey)
        xor_string = chr(random_key_value ^ rs_integer_flag_values[i])
        dec_flag += xor_string

    if 'flag{' in dec_flag:
        flag = re.search(r"flag\{(.+?)\}$", dec_flag)
        print(flag.group(0))
        break
    
    seed_val = seed_val + 1
    if seed_val == '0011': break

```

```
flag{in_the_depths_of_chaos_seeds_of_creation_lie}
```