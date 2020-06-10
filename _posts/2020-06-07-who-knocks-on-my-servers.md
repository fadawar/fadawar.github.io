---
layout: post
title: Who knocks on my servers?
---

**Weird domains and IP addresses in Django‘s exceptions „Invalid HTTP_HOST header“**

Few months back I worked on website for a local [city festival](https://hanusovedni.sk/en/).
It’s popular festival with lots of guests from different countries and a quite long history.
I migrated the site from Wordpress to a brand new Wagtail solution.
We have also moved from sharing hosting to a VPS.
I wanted to progress quickly with development.
So I launched new version of website as soon as possible with just uwsgi responding to all the http traffic. 
Soon after the luanch I started to get weird errors in Sentry:

![Screenshot of error in Sentry](/images/sentry-error-invalid-http-host.png)

Message from the error was:

```
Invalid HTTP_HOST header: 'corpse.xyz'. You may need to add 'corpse.xyz' to ALLOWED_HOSTS.
```

Message was clear.
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
After some time I found the reason behind this strange behavior.
Domains and IP addresses I saw in Sentry or in logs are from HTTP header “Host”.
And _all_ HTTP headers are set by client.
In other words you can’t trust HTTP headers.
Clients can set header to whatever value they like.
For example this is how you can do it in Python with requests library:

```python
import requests

requests.get("http://hanusovedni.sk", headers={"Host": "corpse.xyz"})
```

Those weird requests were coming not from humans but from robots.
There is actually a lot of robots which scan all IP addresses that exist on the Internet
and find for vulnerabilities on websites. There are two groups of such robots:

1. attackers who wants to misuse vulnerabilities
2. white-hat organizations like The Shadowserver Foundation finding and reporting vulnerabilities

## Solution
There is actually no real problem when you see this type of exception.
It means Django is doing it’s job and filters malicious requests.
But we want to filter out such requests as soon as possible.
In most cases it means to filter it at reverse proxy.
In my case it‘s [Traefik](https://traefik.io).
You can create rules for Treafik as labels on your services in `docker-compose.yml`. 
This rule will pass to Django only requests with Host header set to my domain.

```
traefik.http.routers.web-router.rule=Host(`hanusovedni.sk`, `www.hanusovedni.sk`)
```

Full definition of service from `docker-compose.yml`:

```
services:
  web:
    image: ${WEB_IMAGE}
    env_file:
      - secrets.env
    environment:
      DJANGO_SETTINGS_MODULE: "hanusovedni.settings.production"
    volumes:
      - /var/www/static:/static_root
      - /var/www/media:/media_root
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.web-router.rule=Host(`hanusovedni.sk`, `www.hanusovedni.sk`)"
        - "traefik.http.services.web.loadbalancer.server.port=8000"
```
