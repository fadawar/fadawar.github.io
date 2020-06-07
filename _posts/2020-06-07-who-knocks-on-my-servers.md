---
layout: post
title: Who knocks on my servers?
---

**Weird domains and IP addresses in Django‘s exceptions „Invalid HTTP_HOST header“**

Few months back I worked on a website for a local [city festival](https://hanusovedni.sk/en/).
It’s one of those bigger ones with lots of guests from foreign countries and a quite a long history.
The solution moved from Wordpress to a brand new Wagtail website.
We have also moved from sharing hosting to VPS.
I wanted to progress quickly with development =>
I launched new version of website as soon as possible with just uwsgi responding to all the http traffic. 
Soon after the luanch I started to get weird errors in Sentry:

![Screenshot of error in Sentry](/images/sentry-error-invalid-http-host.png)

The message was clear:

```
Invalid HTTP_HOST header: 'corpse.xyz'. You may need to add 'corpse.xyz' to ALLOWED_HOSTS.
```

There were requests coming to my server from domain which is not allowed.
How it can be?
_“Maybe someone pointed his domain to my server’s IP address by mistake?”_ was first thought in my head.
But there were many of those requests from many different domains.
And they tried to access weird URLs like:

```
/wp-login.php
/wp/wp-login.php
/cgi-bin/test.sh
/cgi-bin/hello.sh
```

And also some of them came not from domain names but from IP addresses.
That was even more weird.

## You can’t trust Host header
There is a reason behind this strange behavior.
Domains and IP addresses I saw in Sentry or in logs are from HTTP header “Host”.
And _all_ HTTP headers are set by client.
In other words you can’t trust HTTP headers.
Simple reason - anybody can set it to whatever values he/she likes.
This is how you can do it in Python with requests library:

```python
import requests

requests.get("http://hanusovedni.sk", headers={"Host": "corpse.xyz"})
```

Those weird requests were coming not from humans but from robots.
There is actually a lot of robots scanning all the IP addresses existing on the Internet
and finding vulnerabilities on websites. There are two groups:

1. attackers who wants to misuse vulnerabilities
2. white-hat organizations like The Shadowserver Foundation finding and reporting vulnerabilities

## Solution
There is no real problem here – Django is doing it’s job and filters malicious requests.
But we want to filter out such requests as soon as possible.
In most cases it means to filter it at reverse proxy.
In my case it‘s Traefik.
I set a rule on my service to pass to Django only requests with Host header set to my domain.

```
traefik.http.routers.web-router.rule=Host(`hanusovedni.sk`, `www.hanusovedni.sk`)
```
