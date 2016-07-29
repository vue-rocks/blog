---
layout: post
title:  "Socket.io with AngularJS boilerplate"
description: "In this post we'll be looking at a smallest possible socket.io chat application to get you started."
date:   2015-05-29 13:18:15+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

[Socket.io][socketio] has truly been a godsend to making realtime applicaions. What used to require writing complex code, ajax polling and other tricks of the trade is now a few lines of code. And it's fast too - in fact we have a situation where client sends a POST request to python server, which in turn is passed to node server via [redis][redis] [pub/sub][redis-pubsub] and that in turn send a message to the client with socket.io. The interesting bit is that although the socket.io round-trip involves more components, it delivers the message back to client before the response is received. While pure HTTP implementation with network latency in the mix might win the race, it is still a valid contender.

I often find myself experimenting on some concept where I need a quick and simple boilerplate to get me going - setting it all up takes time after-all. There are great demos out there, but they often cover more ground than necessary, what I am mostly interested in is the bare minimum working backbone that I can build upon. So here is a stripped-down chat application in just 2 files, doesn't even contain an http server!

## Server.js


```javascript
var io = require('socket.io').listen(3000);

io.on('connection', function(socket){
  socket.on('message', function(msg){
    io.emit('message', msg);
  });
});

console.log("Server up!");
```

## Index.html

Since Angular is not aware of socket receiving a message, we need to wrap the `$scope` update in an `$apply` function so that angular would pick it up.


```html
<!doctype html>
<html ng-app="myApp">
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.3.5/socket.io.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.3.15/angular.min.js"></script>
  </head>
  <body ng-controller="myAppCtrl">
    <textarea ng-model="messages" rows=15 cols=80></textarea><br/>
    <input ng-model="message"></input>
    <button ng-click="send()">Send</button>
    <script>
      var app = angular.module("myApp", []);
      app.controller("myAppCtrl", function($scope) {
        var socket = io('http://localhost:3000');
        $scope.messages = '';
        $scope.send = function(){
          socket.emit('message', $scope.message);
          $scope.message = '';
        }
        socket.on('message', function(message){
          $scope.$apply(function(){
            $scope.messages += message + "\n";
          });
        });
      });
    </script>
  </body>
</html>
```

## Setup


```bash
npm install socket.io
node server.js
```

[socketio]: http://socket.io
[redis]: http://redis.io
[redis-pubsub]: http://redis.io/topics/pubsub
