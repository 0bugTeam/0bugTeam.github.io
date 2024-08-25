---
title: a little bit of tomcroppery
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Misc
draft: false
---

## Description

Once you crop an image, the cropped part is gone... right???

Attachment : 

![image-attachment](/images/litctf/image.png)

## Solution

After trying exiftool, binwalk and other stegnography tool and polygot methods, i didn't find anything. Then `Zacian` told me about the possibility of an image inside an image.

In that case the binary data should persist or exist with the given data. We can check this by using `xxd image.png | grep PNG`

This shows us the following

```
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
0001b180: 504e 475b bc5f c83b 2d47 5168 3de7 a1e3  PNG[._.;-GQh=...
0006db70: 6082 8950 4e47 0d0a 1a0a 0000 000d 4948  `..PNG........IH
```

hmm more than one PNG header, now this is not normal for a PNG image, we can verify this with other normal PNG image, which only contains one PNG header and one IEND

We can extract the other PNG file data using `dd`

First we need to get the starting address of PNG header which is `0006db70`, this is in hex we have to convert it into decimal 

```bash
python -c 'print(0x0006db70 + 2)' # 449394
```

2 is to skip the '\`' in the image data and start from 82 instead of 60. Now we can use dd to skip `449394` bytes infront of it basically setting the starting offset

```bash
dd if=image.png of=flag.png bs=1 skip=449394
```

And we get our flag

![image-attachment](/images/litctf/flag.png)

```
Flag : LITCTF{4cr0p41yp5e_15_k1nd_0f_c001_j9g0s}
```

We used Text From Image [extraction tool](https://brandfolder.com/workbench/extract-text-from-image) but it outputs wrong data, so make sure to fix it 

## Source of Knowledge

A CTF Challenge Writeup that shows how dd can be used to extract the image
https://afnom.net/wtctf/2019/magic/, this is the main source that helped me

Althought not entirely related by, i rembered this video while solving this challenge
https://youtu.be/AMMOErxtahk : STOP WASTING YOUR TIME AND LEARN MORE HACKING!
