---
title: PackedAway 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Reversing
draft: false
---

```
PackedAway

To escape the arena's latest trap, you'll need to get into a secure vault - and quick! There's a password prompt waiting for you in front of the door however - can you unpack the password quick and get to safety?
```

After downloding and extracting the binary, if we run `strings` on it, we can see two imporant things

one is that, we get to see part of the flag

```
Hr3t_0f_th3_p45}
```

and other is a message whcih will help us

```
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.22 Copyright (C) 1996-2024 the UPX Team. All Rights Reserved. $ 
```

so the binary is obfuscated or packed with a tool named `upx`, we can easily *unpack* this by using a tool in kali linux named `upx`

```bash
upx -d packed

# -d for decompress
```

if it works it will say `Unpacked 1 file.`. Now we can ring strings again to get the flag

```bash
strings packed | grep "HTB"
```

XD, we can see that there are decoys to make it more complex, if we ever tried it through other debugging softwares.

----- 

Source Help : https://youtu.be/0jVikfySiII