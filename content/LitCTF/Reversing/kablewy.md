---
title: kablewy
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Reversing
draft: false
---

## Description

WARNING: your computer might go kablewy if you try to do this one... URL: http://litctf.org:31782/

## Solution

So as it says, yes opening the link in browser somewhat crashes it. so we might have to just find the cause of the crash in the first place

curling the URL gives us the following

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + TS</title>
    <script type="module" crossorigin src="/assets/index-DLdRi53f.js"></script>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

we get to see a JS file `/assets/index-DLdRi53f.js`. After downloading the file using wget or curl, we can find a packed up JavaScript code

To understand the code properly i used [beautifier.io](https://beautifier.io/) to beautify or format the code and i found different base64 encoded strings 

After saving the output to a file named `beautified.js`, i used some simple tools like grep and awk with regex to decode and output the flag

```bash
cat beautified.js | grep -oE "K\S+K" | base64 -d | grep -oE "d\S+7" | base64 -d | grep -oE "'\S{1}'" | tr -d "'"  | awk '{printf "%s", $0}'
```

```
Flag : LITCTF{k3F7zH}
```


