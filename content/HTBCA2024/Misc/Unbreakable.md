---
title: Unbreakable 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Misc
draft: false
---

## Description

Unbreakable

Think you can escape my grasp? Challenge accepted! I dare you to try and break free, but beware, it won't be easy. I'm ready for whatever tricks you have up your sleeve!

## Solution

Well for this challenge i had to sell my soul for a bit =]

The challenge comes with a netcat instance and a downlaodable python file

```python
blacklist = [ ';', '"', 'os', '_', '\\', '/', '`',
              ' ', '-', '!', '[', ']', '*', 'import',
              'eval', 'banner', 'echo', 'cat', '%', 
              '&', '>', '<', '+', '1', '2', '3', '4',
              '5', '6', '7', '8', '9', '0', 'b', 's', 
              'lower', 'upper', 'system', '}', '{' ]

while True:
  ans = input('Break me, shake me!\n\n$ ').strip()
  
  if any(char in ans for char in blacklist):
    print(f'\n{banner1}\nNaughty naughty..\n')
  else:
    try:
      eval(ans + '()')
      print('WHAT WAS THAT?!\n')
    except:
      print(f"\n{banner2}\nI'm UNBREAKABLE!\n") 
```

Looking at the file we can see that it's a pyjail type of challenge.

So basically after a lot of trial and error, where at first i was trying to get a shell but then i remembered it is just about reading the file. Which got me the following payload to bypass the blacklist restriction.

```python
print(open('flag.txt').read())
```

*Although the value is continated with `()`, it runs and prints the flag and then executes the exception part*


## Source of knowledge

https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes\
https://docs.python.org/3/library/functions.html