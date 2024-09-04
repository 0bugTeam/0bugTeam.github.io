---
title: TimeKORP 
date: 2024-03-14
author: 1byteBoy
tags:
  - Web
draft: false
---

## Description

TimeKORP 

## Solution

In the provided downloadable source code file, the `TimeModel.php` file has some interesting piece of code

```php
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->command = "date '+" . $format . "' 2>&1";
    }

    public function getTime()
    {
        $time = exec($this->command);
        $res  = isset($time) ? $time : '?';
        return $res;
    }
}
```

It seems it's executing the `date` os command as 

```bash
date '+"%Y-%m-%d %H:%M:%S"' 2>&1
```

So after playing around with it for a bit, i discovered that it is vulnerable to OS command injection vulnerability. Where we can provide a payload like the following

```bash
"'&&cat /flag # 
```

better to url encode it `"'%26%26cat+/flag+%23`, which will read the flag from `/flag`.

Note: we can know the path from the given Dockerfile