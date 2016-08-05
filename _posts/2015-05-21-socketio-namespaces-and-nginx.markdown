---
layout: post
title:  "Proxy Custom Socket.IO Namespaces with NGINX"
description: "Proxying custom Socket.IO namespaces with NGINX is not straightforward. Hopefully this will save time for the next person attempting it."
date:   2015-05-21 22:30:00+02:00
size: 12
sizeDesktop: 6
textColor: grey-50
metaColor: grey-600
comments: true
author: justas
---

A while ago I implemented a chat server based on [Socket.IO][socketio] for [hopper.dk][hopper]. In addition to the chat server, the system already had 2 HTTP servers: [Gunicorn][gunicorn] for our RESTful API and [NGINX][nginx] for serving [AngularJS][angular] frontend. To make SSL management easier I decided to put all 3 servers behind a single NGINX host. It all went pretty smoothly except for 1 thing- my chat server with a custom namespace was unreachable through NGINX.

Let's illustrate this with some code

## Socket.IO Server


```javascript
io = require('socket.io')();
io.listen(3000);

var myNamespace = io.of('/my-namespace');
myNamespace.on('connection', function(socket){
  console.log('someone connected');
  socket.emit('news', 'Hi there!!');
  myNamespace.emit('news', 'Someone joined the party!');
});
```

## NGINX Host


```bash
server {
        listen 8080;

        server_name localhost;

        location ~/(my-namespace).*$ {
          proxy_pass         http://127.0.0.1:3000;
          proxy_http_version 1.1;
          proxy_set_header   Upgrade    $http_upgrade;
          proxy_set_header   Connection "upgrade";
        }
}
```

## Connection attempts fail with HTTP 404

![Client connection attempts fail][fail-img]

This setup results in Socket.IO client trying to establish a connection to the server with no success. After a bit of furious *DuckDuckGoing* and *Googling*, I gave up and decided to look for answers in [Socket.IO's code][socketio-gh].

The solution is simple: in addition to proxying your custom namespace, you also have to proxy `/socket.io` resource to the Socket.IO server:


```bash
server {
        listen 8080;

        server_name localhost;

        location ~/(my-namespace|socket\.io).*$ {
          proxy_pass         http://127.0.0.1:3000;
          proxy_http_version 1.1;
          proxy_set_header   Upgrade    $http_upgrade;
          proxy_set_header   Connection "upgrade";
        }
}
```

## Success

![Client connection successfully established][success-img]



I hope this helps.

[socketio]: http://socket.io/
[socketio-gh]: https://github.com/Automattic/socket.io
[hopper]: https://hopper.dk
[gunicorn]: http://gunicorn.org/
[nginx]: http://nginx.org/
[angular]: https://angularjs.org/
[fail-img]: {{ site.baseurl }}/assets/socketio-namespaces-and-nginx/fail.png
[success-img]: {{ site.baseurl }}/assets/socketio-namespaces-and-nginx/success.png
