---
title: hide and split
date: 2023-12-10T13:37:49+02:00
description: Writeup for hide and split [TCP1P]
author: 1byteBoy
tags:
  - Forensics
draft: false
---

**Author**: `underzero`

Explore this disk image file, maybe you can find something hidden in it.

-----

With this challenge we get a zip file containing a file named `challenge.ntfs`.
One googling we don't get anything nor does any AI model knows about this file format, so this is a fake file format, but if we do run file command on it we see the following

```
challenge.ntfs: DOS/MBR boot sector, code offset 0x52+2, OEM-ID "NTFS    ", sectors/cluster 8, Media descriptor 0xf8, sectors/track 0, dos < 4.0 BootSector (0x80), FAT (1Y bit by descriptor); NTFS, sectors 10239, $MFT start cluster 4, $MFTMirror start cluster 639, bytes/RecordSegment 2^(-1*246), clusters/index block 1, serial number 01fe3c7444d0702ed
```

so i tried to look for what `DOS/MBR boot sector` means

**According to Goolge**
*A master boot record (MBR) is a special type of boot sector at the very beginning of partitioned computer mass storage devices like fixed disks or removable drives intended for use with IBM PC-compatible systems and beyond. The concept of MBRs was publicly introduced in 1983 with PC DOS 2.0.*

Anyways we don't need all this, and we don't need any technical things with this forensics challenge, no need of any emulator or VMs, all we have to do is string the file and get the hex values and then convert them into the a PNG format or PNG file 

```
....
Unfortunately this is not the flag
The flag has been split and stored in the hidden part of the disk
21871c72c821871c72c821871c72c821871c72c821871c72c821871c72c821871c72c821871
FILE0
Unfortunately this is not the flag
The flag has been split and stored in the hidden part of the disk
c72c821871c72c821871c72c821871c72c8b929e7c9af2cc6f6ec8d07c791a1dc5ad8ad7384
FILE0
Unfortunately this is not the flag
The flag has been split and stored in the hidden part of the disk
1c72c821871c72c821871c72c821871c72c821871c72c821871c72c821871c72c821871c72c

......

c49e448e448e448e4487faf7fd99fb41dd95101ee0000000049454e44ae426082
```

most of the hex string are in like this, if we calculate the length of it it's 75 in length, all of them except the last one which 64 in length

now we can save the output in a file and parse the hex string using regex and save them in a file and then copy it to decode 

```
strings challenge.ntfs > encoded.txt
```

Here is a code that i used to parse the strings

```python
import re

# Define the regular expression pattern
pattern = r'[0-9a-f]{64,75}'

# Input and output file names
input_file = 'encoded.txt'
output_file = 'output.txt'


# Read the content of the input text file
with open(input_file, 'r') as file:
	file_content = file.read()

# Use re.findall to find all matching strings in the content
matches = re.findall(pattern, file_content)

# Append the matched strings to the output text file on the same line
with open(output_file, 'a') as out_file: # Open in 'append' mode
	for match in matches:
		out_file.write(match)

# Optionally, add a newline character at the end of the file
with open(output_file, 'a') as out_file:
	out_file.write('\n')
```

we can copy the output and paste in a decode https://gchq.github.io/CyberChef/ and it will automatically decode it from Hex , if not first decode it from Hex and then use Render Image to get the QR Code 

we can use any online QR Code scanner to scan image file and get the flag
https://scanqr.org/

```
TCP1P{hidden_flag_in_the_extended_attributes_fea73c5920aa8f1c}
```

*Note i used https://regexr.com/, to make my regex*



