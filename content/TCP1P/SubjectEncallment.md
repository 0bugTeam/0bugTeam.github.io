---
title: Subject Encallment
date: 2023-12-10T13:37:49+02:00
description: Writeup for Subject Encallmente [TCP1P]
author: 1byteBoy
tags:
  - Reversing
draft: false
---

**Author**: `Kisanak`

If there's something strange. In your neighborhood. Who you gonna call?

-----------

The challenge came with a 64-bit ELF executable, first thing i did is open it up with Ghidra. Looking at the main function we can see the following

```c
undefined8 main(void)

{
  puts("=====FLAG-GENERATOR-INATOR-3000=====");
  sleep(2);
  secretFunction();
  sleep(2);
  return 0;
}
```

so having a little knowledge of reverse engineering or reversing in general or if we study about it a bit we will know that, we can manipulate the `call` or *jump* function or functionality to call other functions even though it is never called in the main function

this can be done by various common methodologies like settings up breakpoints, jumping to different function address or changing the return address.  

I did try different ways, but it was not working and i was struggling to understand but after a bit a article helped me out

first of all this are the main functions

```
0x00000000000011a9  secretFunction
0x000000000000133b  phase1
0x00000000000013f2  phase2
0x00000000000014af  phase3
0x000000000000159b  phase4
0x000000000000161b  phase5
0x00000000000016e0  phase6
0x00000000000017a1  phase7
0x0000000000001804  phase8
0x000000000000188f  phase9
0x000000000000194b  phase10
0x00000000000019a1  phase11
0x00000000000019e3  phase12
0x0000000000001a7b  phase13
0x0000000000001af7  phase14
0x0000000000001ba5  printFlag
0x0000000000001e22  main
```


**Now according to the article**
*In order to modify the application at runtime, it is necessary to run the program and then stop it again before it finishes cleanly. The most common way is to use a breakpoint  (_help break_) on main:*

Now doing the following will give us the flag

```
$ gdb chall
(gdb) break main
Breakpoint 1 at 0x1e26
(gdb) run
Starting program: /mnt/workspace/LiveCTFs/TCP1P/RE/chall 
[Thread debugging using libthread_db enabled]                                                                                                         
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, 0x0000555555555e26 in main ()
(gdb) jump printFlag 
Continuing at 0x555555555ba9.
TCP1P{here_my_number_so_call_me_maybe}
[Inferior 1 (process 225332) exited with code 01]
(gdb)
```

source of knowledge 
https://www.sans.org/blog/using-gdb-to-call-random-functions/

*There might be other really good ways of doing it but it worked for me*

`TCP1P{here_my_number_so_call_me_maybe}`