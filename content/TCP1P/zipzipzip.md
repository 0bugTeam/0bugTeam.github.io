---
title: zipzipzip
date: 2023-12-10T13:37:49+02:00
description: Writeup for zipzipzip [TCP1P]
author: 1byteBoy
tags:
  - Misc
draft: false
---

## Description

**`Author: botanbell`**

unzip me pls

## Intuitio

OK so this challenge was pretty straight forward but tricky a resource and time intensive, all we have to do is extract the zip files one after another until the last one, so i used chatGPT to write a simple python script that automate reading password from password.txt and unzipping the file one after another.

The initial file was named `zip -25000.zip` after extracting with the password from the password.txt it gives us a new zip file named `zip-24999.zip` and a new password file

so it's likely that it will decrement upto `zip-1.zip` and maybe give us the flag file. So i wrote a python script but it was slow and resource intensive, i left and tried other challenges but then my friend showed me his python script as it was not working after extracting some files, but what got me interested is that he is using system commands instead of fully utilizing python functionality, so then and then i thought of using bash or shell scripting to make the process and it was rally fast and resource friendly

Here is my script

## Solution

```bash
#!/bin/bash

initial_zip_number=25000
password_filename='password.txt'

function unzip_file {

	local zip_filename="$1"
	local password="$2"

	if [[ -f "$password_filename" ]]; then
		unzip -o -P "$password" "$zip_filename"
		rm "$zip_filename"
	else
		echo "Password file '$password_filename' not found."
		exit 1
	fi
}

while true; do

	zip_filename="zip-$initial_zip_number.zip"
	
	if [[ -f "$zip_filename" ]]; then
		echo "[*] $zip_filename"
		password=$(tr -d '\r' < "$password_filename") # Remove any carriage returns
		unzip_file "$zip_filename" "$password"
		initial_zip_number=$((initial_zip_number - 1))
	else
		echo "No more zip files found."
		break
	fi
done
```

after couple of minutes of running, it got to the last zip and extracted a file named *flag.txt* which contains the flag

```
TCP1P{1_TH1NK_U_G00D_4T_SCR1PT1N9_botanbell_1s_h3r3^_^}
```
