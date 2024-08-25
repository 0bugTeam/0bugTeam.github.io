---
title: anti-inspect
date: 2023-12-10T13:37:49+02:00
author: zacian
tags:
  - Web
draft: false
---

## Description

can you find the answer? WARNING: do not open the link your computer will not enjoy it much. URL: http://litctf.org:31779/ Hint: If your flag does not work, think about how to style the output of console.log

Author : halp

## Solution

Simply curl the given curl and it will print the flag

```html
$ curl http://litctf.org:31779/

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <script>
      const flag = "LITCTF{your_%cfOund_teh_fI@g_94932}";
      while (true)
        console.log(
          flag,
          "background-color: darkblue; color: white; font-style: italic; border: 5px solid hotpink; font-size: 2em;"
        );
    </script>
  </body>
</html>
```