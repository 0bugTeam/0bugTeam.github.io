---
title: Guess My Number
date: 2023-12-10T13:37:49+02:00
description: Writeup for Guess My Number [TCP1P]
author: 1byteBoy
tags:
  - Misc
draft: false
---

**Author**: `rennfurukawa`

My friend said if i can _guess_ the right number, he will give me something. Can you help me?

`nc ctf.tcp1p.com 7331`

The challenge came with a ELF binary file, since it is not a BE challenge i had not considering to run `checksec` or use gdb, instead i directly opened it with `ghidra auto guess`

looking at the **vuln** function we can see the following code

```c
void vuln(void)

{
  int iVar1;
  
  key = 0;
  srand(0x539);
  iVar1 = rand();
  printf("Your Guess : ");
  fflush(stdout);
  __isoc99_scanf(&DAT_001020cb,&key);
  if ((key ^ iVar1 + 0x1467f3U) == 0xcafebabe) {
    puts("Correct! This is your flag :");
    system("cat flag.txt");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("Wrong, Try again harder!");
  return;
}
```

The code is pretty straight forward,  first the key is set to 0 and it is using a function named `srand` , simply said this function sets a seed for the rand() function to generate a random number based on the seed, so since we know the seed value, we also can get the random number

next is where we input, the input is will be integer number taken into key variable `__isoc99_scanf(&DAT_001020cb,&key);`

the last and main is the condition statement, which gives us hint and helps to solve us the challenge. 

`if ((key ^ iVar1 + 0x1467f3U) == 0xcafebabe)`

let's understand this in simple math

```
$ python3
>> key   = 2
>> iVar1 = 3
>> some_value = 4
>> # now if we do
>> key ^ iVar1 + some_value
>> 10
```

here 10 is alternative to `0xcafebabe` , so we know iVar1 which is random value, which can be generated using the seed, we know *some_value* which is `0x1467f3U` and we know the output which is `0xcafebabe`

and now if we do say `10 ^ 3 + 4` we get 2 or the key. Here is a C code that can give us the key value 

```c
#include <stdio.h>

int main () {
	int rand_value;
	int key = 0;
	srand(0x539);
	rand_value = rand();
	// 0xcafebabe converted into decimal is 3405691582 
	key = 3405691582 ^ rand_value + 0x1467f3U;
	printf("Key is : %d", key);
}
```

`key is : -612639902` and now if we enter it we get our flag

```
=======              WELCOME TO GUESSING GAME               =======
======= IF YOU CAN GUESS MY NUMBER, I'LL GIVE YOU THE FLAG  =======

Your Guess : -612639902
TCP1P{r4nd0m_1s_n0t_th4t_r4nd0m_r19ht?_946f38f6ee18476e7a0bff1c1ed4b23b}
Correct! This is your flag :
```

source of help and knowledge
https://www.taintedbits.com/2020/06/07/binary-exploitation-pwnable-kr-level-6/


