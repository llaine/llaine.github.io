---
layout: post
title: "Introduction to webSockets with NodeJS"
date: 2014-10-12 12:12:09 +0200
comments: true
categories: JavaScript NodeJS webSockets
description: "Introduction to webSockets"
keywords: "miage, bordeaux, webSockets, louis lainé, lainé, louis, javascript, nodejs, github, angularjs, ecmascript6"
---


# Introduction

Today a small article about a new web standard, the webSockets.

This new protocol allow to create a bi-directional connexion between the client and the server in order to solve some HTTP problems and his uni-directional connexion. 

You can easily create real time application as games, chat and so. 

The server is able to "push" informations on the client even if the client hasn't asked for.   

# Use case

We are going to create a simple chat. 

- The user can send messages to connected clients with a username from a web page
- The user can send messages to connected clients from console.
- The server is able to count the number of connected client and trace the conversation. 

# Tree directory 

```bash Init the repository
$ mkdir webSocket && cd webSocket ; npm init
```

We are going to use a simple library called `websocket` which is the (mostly) pure JavaScript implementation of the WebSocket protocol versions 8 and 13 for Node. 

```json package.json
"dependencies": {
    "websocket": "^1.0.8"
} 
...
```


# Server

Copy paste this code sample in a server.js file. 


```javascript server.js

var WebSocketServer = require('websocket').server
  , http = require('http')
  , lesConnexions = [] // Will store all the connection
    // process HTTP request. Since we're writing just WebSockets server
    // we don't have to implement anything.
  , server = http.createServer()
  , PORT = 1337;


server.listen(PORT);

/* bootstraping the HTTP server */
wsServer = new WebSocketServer({
    httpServer: server
});

log("Server listenning on " + PORT + " ...");

/* when a new client is trying to connect on the server */
wsServer.on("request", function(req, res){
    var connection = req.accept(null, req.origin);

    // add the new connection in the current connection
    lesConnexions.push(connection);     

    connection.on("message", function(message){
        // When a message is send
    });

    connection.on("close", function(){
        // When the connection is closed
    });


});

/**
* Log into the NodeJS console
* @item : the item to be logged
*/
function log(item){
    console.log("[" + new Date().toDateString() + "] " + item);
}
```

This is the basic implementation of the server with websocket package. 
Whe have here a server listenning to the __1337__ port. 


# Client 

## Console 

The websocket library allow you to easily implement a client in a simple console app. 

```javascript client.js
var WebSocketClient = require('websocket').client;


var client = new WebSocketClient();

client.on('connectFailed', function(error) {
    console.log('Connect Error: ' + error.toString());
});

client.on('connect', function(connection) {
    console.log('WebSocket client connected');

    connection.on('error', function(error) {
        console.log("Connection Error: " + error.toString());
    });

    connection.on('close', function() {
        console.log('echo-protocol Connection Closed');
    });

    connection.on('message', function(message) {
        // when a message is send
    });
    
});

client.connect('ws://127.0.0.1:1337/');
```


## Browser version

We also want to discuss through the tiny chat from a internet browser so let's create simple index.html file. 

__Note__ : I use bower to manage my libraries, in this case it's only jQuery. 

```html 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WebSocket</title>
</head>
  <body>
    
    <div id="head">
      <span id="state"></span>
      <input type="text" id="name" placeholder="Pseudo" value="webUser">
    </div>
    
    <div id="chatMessages">
      <input type="text" id="input">
      <ul id="messages"></ul>
    </div>
    
    <script src="bower_components/jquery/dist/jquery.js"></script>
    <script>
      // The client javascript implementation
    </script>
  </body>
</html>
```

Related to the html file, we need to use the simple API to connect to our server. 


```javascript 

/**
* Send a message with a username
* @content : the content of the message
* @username : the username of the message
*/
function sendMessage(content, username){
  
  if(!content || !username) return;
  
  var msg = {
    txt : content,
    username : username
  };
  
  connection.send(JSON.stringify(msg));
  
}

(function($){
    var state = $("#state")
    ,	input = $("#input")
    ,	messages = $("#messages")
    ,	username = $("#name");

    window.WebSocket = window.WebSocket || window.MozWebSocket;

    var connection = new WebSocket('ws://127.0.0.1:1337');

      connection.onopen = function () {
          // connection is opened and ready to use
          state.html("open");
      };

      connection.onerror = function (error) {
        // an error occurred when sending/receiving data
          state.html("error ");
          input.prop('disabled', true);
          username.prop('disabled', true);
      };

      connection.onmessage = function (message) {
        // message is send

      };

      
      input.keydown(function(e){
      
        /* when the enter key is pressed */
        if(e.keyCode === 13){

          var theMsg = $(this).val();
          
          sendMessage(theMsg, username.val());
          
          $(this).val("");

        }
        
      });


  })(jQuery);


```


Now let's look at how to manager the sending and receiving of messages. 


### Server

```javascript 

  // In the request callback. 
  
  log("connection from " + connection.remoteAddress);
  log("currently " + lesConnexions.length + " clients");

 connection.on('message', function(message) {
    // We only wants to send char, not bytes.
    if(message.type === "utf8"){
      
      // for each connected clients
      for (var i = lesConnexions.length - 1; i >= 0; i--) {
        lesConnexions[i].sendUTF(message.utf8Data);
      }
 
    }
 
  });
 
  connection.on('close', function(connection) {
    lesConnexions.splice(lesConnexions[lesConnexions.length], 1);
    log("connection closed");
  });


```

### Client - console

The console implementation will be a little bit different because we need to get the user inputs. 

I'm going to use the __sys__ library. 

```javascript client.js

var sys = require("sys");
var stdin = process.openStdin();
// ...

 connection.on('message', function(message) {
     if (message.type === 'utf8') {
         var msg = JSON.parse(message.utf8Data);
         if(msg.username === undefined) msg.username = "default";
 
         console.log("--> [" + msg.username + "] : "+  msg.txt  +"");
     }
 });
 
 // add callback for user inputs
 stdin.addListener("data", function(d) {
     // note:  d is an object, and when converted to a string it will
     // end with a linefeed.  so we (rather crudely) account for that
     // with toString() and then substring()
     var msg = d.toString().substring(0, d.length-1);
 
     if(connection.connected || msg){
         connection.sendUTF(JSON.stringify({
             txt : msg,
             username : "console"
         }));
 
     }
 });
```


### Client - index.html 

```javascript index.html
 connection.onmessage = function (message) {
  var value = JSON.parse(message.data);
 
 
  messages.append(
      "<li>" 
        + new Date(message.timeStamp).toDateString() 
        + " | <strong>" + value.username 
        +  "</strong> - <pre> " + value.txt 
        + "</pre>
      + "</li>"
    );

  };

```



# Conclusion 


We seen how to deal with webSockets through those simple use cases. 

You can find all the code we've seen on [github](https://gist.github.com/llaine/2072f26c3dcde373a141)

Feel free to comment for any informations! 








