---
title: Elite
date: 2024-08-26
author: zacian
tags:
  - Web
draft: false
---


## Description

Do you think you are ELITE? If you really believe so...you are more than welcome to join our Elite Club. However, we will decide whether or not you are worthy enough to join us. All the best!

Check out our Website here: http://chall.ycfteam.in:6375/

## Solution

Visiting the URL through browser for the first time loads a page that says **Are you ready to join the Elite Club?** with a button at the bottom of the page that says `Join the Elite Club`. Clicking the button takes to `http://chall.ycfteam.in:6375/elite` where in the page it says Only Elite Agents can access this page.

So it's hinting us with `User-Agent` header, as of previous experience i quickly understood that i have to add or change some headers, so i took the request to burpsuite repeater

In the following i will change or add header based on the response and explain through it.

For the first one let's change the user-agent value to `Elite`

```
User-Agent: Elite
```

----

and we get the following reponse

```
Elite members can access this endpoint only via our dedicated proxy chain:
50.23.41.34
3.54.85.90
110.34.87.34
10.43.21.25
```

At first i thought i had to use proxychains but later realised there is http header where we can provide proxy server address thorugh which we want our request to go through. It's called [X-Forwarder-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)

```
X-Forwarded-For: 50.23.41.34, 3.54.85.90, 110.34.87.34, 10.43.21.25
```

----

After sending the request with these proxies in place we get the following response back

```
Nope! We only accept requests from our Elite port number - 31173. Lea
```

Now like the previous header there is also a similar header called [X-Forwarded-Port](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/x-forwarded-headers.html#x-forwarded-port), we don't get to see it listed in MDN Docks but if we google based on the response we get to find this header 

```
X-Forwarded-Port: 31173
```
----

This got us the following response

```
Wait! Where did this request even originate from? How dare you try to enter our club.
```

Now if we simply google with the keyword `originated`, something like `originated header` we can find details about a header called [Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) 

```
Origin: http://chall.ycfteam.in:63753
```
----

after sending the request with Origin header with all other before it, we get the following response

```
Something's fishy... Your request to join the elite club should have been cached in each proxy server for 5 seconds. I don't like this. I can't allow you to join. Be gone!
```

Like before if we do simple googling with time and headers for http request, we can find a header called [Age](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Age), this request mainly tells how long the request should hold or cache in the proxy server 

Now we have 4 proxy server, and we need to cache the request for 5 sec, so 4 * 5 is 20 sec

```
Age: 20
```

----

This give us response as

```
Oops! It seems you are too late my friend... We already closed the club registration on 27th May 2024 at 11 AM IST. Maybe next time...See ya!
```

If you don't know a simple google search will help you but basically we can specify the date a request has been originated using the [Date](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date) header. If proper checks are not done, we can use it to bypass many things time related in a website 

So the date should be before 27th May

```
Date: Wed, 21 Oct 2015 07:28:00 GMT
```

Sending this request finally gives us a reponse with the flag in it

```
Alright then! You've proven that you are indeed Elite!! Congratulations on joining the club! It's great to have you on board with us. Here's your exclusive welcome gift: 
CyGenixCTF{W3lc0me_t0_Th3_ELIt3_5qU4d_5bf90dac2b7}
```

----

This challenge was great to learn about different HTTP headers and how they work\
[developer.mozilla.org](https://developer.mozilla.org/) helped a lot