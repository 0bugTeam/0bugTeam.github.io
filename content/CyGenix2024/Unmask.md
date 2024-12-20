---
title: Unmask
date: 2024-08-26
author: 1byteBoy
tags:
  - Forensics
draft: false
---

## Description

A crucial file has been tampered with in a way in order to conceal its true contents. Your mission is to unmask the data and retrieve the hidden information. Lets find out your worth and if you have what it takes to uncover and extract the forensic evidence. And if you do, I assure you, the return will be rewarding. All the best Agent!

## Solution

The challenges have us a file that contains some hex data, so i copied it and put it directly into [CyberChef](https://gchq.github.io/CyberChef/) inorder to try decoding it from hex and found the following

```
RDHI
   


GNP)÷
```

it's PNG and IHDR in reverse, now looking at the given file and it's data i found out that the hex string is being reversed

```
# Reversed
52 44 48 49 0d 00 00 00 0a 1a 0a 0d 47 4e 50 89

# Original
89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52
```

so i explain chatGPT and asked it to write me a simple script to reverse the values in each line and in the end convert the hex to binary so that i can get the image file

```py
# Function to reverse bytes in each line
def reverse_bytes_in_line(hex_line):
    # Remove any spaces or new lines
    hex_line = hex_line.replace(' ', '').strip()
    # Reverse the bytes (two hex characters per byte)
    reversed_line = ''.join(reversed([hex_line[i:i+2] for i in range(0, len(hex_line), 2)]))
    return reversed_line

# Function to convert hex to binary
def hex_to_binary(hex_data):
    # Convert hex string to bytes
    return bytes.fromhex(hex_data)

def process_file(input_file_path, output_file_path):
    with open(input_file_path, 'r') as infile, open(output_file_path, 'wb') as outfile:
        for line in infile:
            reversed_line = reverse_bytes_in_line(line)
            binary_data = hex_to_binary(reversed_line)
            outfile.write(binary_data)

# Paths to your input and output files
input_file_path  = 'challenge.dat'    # Replace with your input file path
output_file_path = 'output_image.bin' # Replace with your output file path

# Process the file
process_file(input_file_path, output_file_path)
```

This gave us the following image

![flag](/images/CyGenix/imageFlag.png)

```
Flag : CyGenixCTF{Th3_jUmbl3D_uP_PNG_3b9cd0e17f}
```