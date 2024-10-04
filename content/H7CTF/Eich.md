---
title: Eich
date: 2024-09-27
author: 1byteBoy
tags:
  - Reversing
draft: false
---

## Description

"Technology is only as good as the people behind it" - Brendan Eich

Author: Abu

## Intuition

The challenge gave us two files, one named `shutthefrontdoor` which seems to contain something random 

```
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[] .....
```

and other named trap with the following data content

```
?_" 2%3]*0NFU▒6ZB>T

6'[P%▒X@▒T6"[5#T 
```

After googling and asking chatGPT, i found that those `[][(![]+[])...`  are actually javascript obfuscated code called `JSFuck` written in esoteric style that resembles the infamous Brainfuck programming language. 

I searched for any online site/source where i can decode the obfuscated code back to it's origial form and found the following site [Decoder-JSFuck](https://enkhee-osiris.github.io/Decoder-JSFuck/)

After de-obfuscating, i used an online Javascript Beautifier to format the code, which finally got me the following code

```js
function A(D, length) {
    let key = '';
    for (let i = 0; i < length; i++) {
        key += D.charAt(i % D.length);
    }
    return key;
}

function B(F, D) {
    let key = A(D, F.length);
    let G = '';
    for (let i = 0; i < F.length; i++) {
        let charCode = F.charCodeAt(i);
        let keyCode = key.charCodeAt(i);
        let GChar = String.fromCharCode(charCode ^ keyCode);
        G += GChar;
    }
    return G;
}
const fs = require('fs');
let C = fs.readFileSync('C.txt', 'utf8');
let D = "6aef677b2c8b645384e713aece4322b045a79f48";
let E = B(C, D);
console.log("Reward: ", E); // b3BlbnlvdXJleWVzYW5kbG9va2F0dGhlbWtleXNwcm9wZXJseQ==
```

The base64 string `b3BlbnlvdXJleWVzYW5kbG9va2F0dGhlbWtleXNwcm9wZXJseQ==` decodes to `openyoureyesandlookatthemkeysproperly` and value for variable D which seems like a hash decrypts to `whatthehelldoyouthinkyouaredoing`

On going through the code for a little bit, we can see that it's simply encrypting data using XOR encryption and the only thing we have to worry about is the key. But after taking a close look at function `A` which is used to create the key, i realised that the logic used simply returns the value of variable `D` which is the hash, and the length of the key depends on the length of the content.

The hash is of length 40, so if the content length it more than 40, the 41st charecter starts from the very first. With that figured out we can write a python script but that won't be necessary, as simply running the js code would do the same.

As for the input, it seems the `trap` file contains the encrypted flag. After decrypting i was getting something not quite readable

```
        >GF$%bPeHz2um.S+k-f']z`99~5nrg(}:{=&*-*66@mg$
```

I thought it might be one more layer of encryption or encoding but sadly it's not. I was confused at what else needed to be done or what i was missig or maybe the content in trap file is just a trap or rabbit hole and was doubting myself, until my other teammate `zacian` came and asked me to change the value of the key, which was a hash to it's decrypted value.

```js
let D = "whatthehelldoyouthinkyouaredoing";
```

and it seems, it did the job and we got our flag `H7CTF{wh@_1N_7h3!r_r1gh7_m1nd_r@v3r$es_j@v4$scr!pt_L0LL!}`

## Source of Knowledge

https://www.trickster.dev/post/javascript-obfuscation-techniques-by-example/
https://www.trickster.dev/post/dont-jsfuck-with-me-part-1/

JSFuck Encoder : https://jsfuck.com/