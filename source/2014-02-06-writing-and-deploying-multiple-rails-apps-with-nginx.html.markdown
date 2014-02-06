---
title: Writing and deploying multiple Rails apps with Nginx
date: 2014-02-06 05:01 UTC
tags: Rails, Nginx, VPS
---

INTRO


### Our Task

Project overview:
http://tutorials.jumpstartlab.com/projects/feed\_engine/feed\_engine.html

### Architecture

```

          |------------|
          |Github OAuth|
          |------------|
             |
          |------|         /  \
          | Auth |  --->  | DB |
          |------|         \  /
|----|  /         \
|User|             |-cookie-|
|----|  \         /
          |-------- |         |-------|       /  \
          |Dashboard|  -gem-> |  API  | ---> | DB |
          |-------- |         |-------|       \  /
                                 ^
                                 |
                                gem
                                 |
                              |--------|      |------------------|
                              |Receiver| <--- | External Services|
                              |--------|      |------------------|

```


#### The Code:

- Authentication
- Frontend
- API
- Callback Receiver
- Gem
- Demo Bot
- Helper Scripts

### Namespacing


### Rack Proxy -> fail


### VPS and Nginx

