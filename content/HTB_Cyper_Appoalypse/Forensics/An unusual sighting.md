---
title: An unusual sighting 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Forensics
draft: false
---

## Description

As the preparations come to an end, and The Fray draws near each day, our newly established team has started work on refactoring the new CMS application for the competition. However, after some time we noticed that a lot of our work mysteriously has been disappearing! We managed to extract the SSH Logs and the Bash History from our dev server in question. The faction that manages to uncover the perpetrator will have a massive bonus come competition!

## Task

This challenge needs us to solve the following questions

## Questions and Answers

**[ ? ] What is the IP Address and Port of the SSH Server**

Answer : we get this answer from the very first connection that we can see in the `sshd.log` file

```
Connection from 100.72.1.95 port 47721 on 100.107.36.130 port 2221 rdomain ""
```

-----

**What time is the first successful Login**

```bash
cat sshd.log | grep "Accepted"
```

The first one from the output is the answer.

-----

**What is the time of the unusual Login**

To get this answer we have to refer a line from the printed description

```
Note: Operating Hours of Korp: 0900 - 1900 
```

That means in 24 hour format it's between 09:00 AM to 19:00 PM 

```bash
cat sshd.log | awk '{print $2}' | cut -d ':' -f1 | sort | uni
```

running the above commnad give us the followiung

```
04
09
10
11
12
13
14
15
17
18
```

we can see that only one number stand out from the time range that is `04`

```bash
cat sshd.log | grep "04"
```

Running the above command will sort the list that has that number and the first outcome of it is the answer.

----


**What is the Fingerprint of the attacker's public key**

we know that the attacker attacked somewhere during 4 AM, so we can run the following command to get the fingerprint of the public key

```bash
cat sshd.log | grep "publickey" | awk '{print $2 $13}' | sed 's/SHA256://' | sort | uniq | grep "04"
```

---

**What is the first command the attacker executed after logging in**

Again using the `04` as a key to the attacker trace will help us

```bash
cat bash_history.txt | grep "04" | head -n 1
```

---

**What is the final command the attacker executed before logging out**

similar as before but now with using tail we get the last ran command

```bash
cat bash_history.txt | grep -P "04:" | tail -n 1
```