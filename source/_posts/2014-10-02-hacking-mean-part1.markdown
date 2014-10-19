---
layout: post
title: "hacking MEAN get started with your own, part 1"
date: 2014-10-02 08:39:51 +0200
comments: true
categories: JavaScript NodeJS AngularJS MongoDb NoSQL
description: "Hacking MEAN stack."
keywords: "miage, bordeaux, louis lainé, lainé, louis, javascript, nodejs, github, angularjs, ecmascript6"
---


# Introduction

The MEAN term refer to a collection of JavaScript based technologies used to develop web applications. 

This article explores the basics of the MEAN stack and show how to create a simple application. 

## MEAN.IO vs you own

There is already a boilerplate of the stack, providing a complete architecture to deal with this kind of application. 
The topic of this article is to show you how to create your own MEAN stack.
It's not a tutorial about [mean.io](http://mean.io/)

## Two parts ? 

The first part of this article is a simple introduction, the second part is a concrete example of the stack. 

# Technologies

- [NodeJS](http://nodejs.org/) is a server side JavaScript execution environment. It’s a platform built on Google Chrome’s V8 JavaScript runtime. It helps in building highly scalable and concurrent applications rapidly.

- [MongoDB](http://www.mongodb.org/) is a schemaless NoSQL database system. MongoDB saves data in binary JSON format which makes it easier to pass data between client and server.

- [Express](http://expressjs.com/) is lightweight framework used to build web applications in Node. It provides a number of robust features for building single and multi page web application. Express is inspired by the popular Ruby framework, Sinatra.

- [AngularJS](https://angularjs.org/) is a JavaScript framework developed by Google. It provides some awesome features like the two-way data binding. It’s a complete solution for rapid and awesome front end development.

# Prerequisites

During all the tutorial i'm on Maverick 10.9.5, follow this tutorial on Unix system will be almost similar.
I use brew to manage the packages. 

### MongoDB 

```Bash

$ brew update 

$ brew install mongodb # Install MongoDB

$ mongod # To run the mongodb server
$ mongo  # To run the mongo cli
```

### NodeJS 

```bash
$ brew install node # Install NodeJS
```

## NPM packages. 

NodeJS come with his own package manager called `npm`.
This package will be helpful to install the modules we need.
Let's see how to install them.

I use the `--save` argument in order to add this module in the `package.json` file, which describe the configuration of our application. 

### Express 

The web framework. 

```bash
$ npm install --save express
```

### Mongoose

Mongoose provides a straight-forward, schema-based solution to modeling your application data and includes built-in type casting, validation, query building, business logic hooks and more, out of the box with NodeJS.

```bash
$ npm install --save mongoose
```

### Yeoman

Yeoman is a generator in order to scaffold a robust and productive base for your project. 
There are a lots of generator. We are going to use the [angular](https://github.com/yeoman/generator-angular)
 one. 

It will be very convenient and usefull to start our front-end part. 

```bash
$ npm install -g --save generator-angular
```
npm will fetch all the modules and install them. 


```bash
$ npm init 
```

This utility will walk you through creating a package.json file.
Enter your information, and then let generate the file.


# Directory structure

You can create your own, i suggest a simple application tree, reusable and flexible.

```bash  the directory structure
$ ls
	back/ 
	front/ 
	node_modules/ 
	package.json 
```

# Back end part

### Server

We are going to create our server. 

```javascript back/server.js
(function(){

    var express = require('express')
      , app = express();

    app.get('/', function(req, res){
        res.send('Hello World ! ');
    });


    app.listen(8081);
})();
```

We have here a simple express instance that listens on 8081 port and sends a string when the route is called. 

Launch the node instance with `node server.js` and fetch the route. 


{% img /images/curl_localhost.png  %}


Ok that's cool, we now have our server. 
Let's organize us by creating folders.   

+ __models__ : This folder will be dedicated to our MongoDb Schema. 
+ __routes__ : this folder will contains all the controller corresponding to routes. 

```bash mean/back/
.
├── server.js 
├── models 
└── routes  
```

### Routing 

Let's first create a route by creating a simple file.

```javascript routes/index.js 
module.exports = (function(){
    'use strict';

    // The express module
    var express = require('express')
      , router = express.Router(); // Routing service of express 

    //Our default route 
    router.get('/', function(req, res) {
        res.send("Hello World from router ! ");
    });
    
    return router;
})();
```
We shift all the routing logic in a dedicated controller. 
This will be more scalable, testable and organized. 

We will no use this route in the `server.js` file. 

```javascript back/server.js
(function(){

    var express = require('express')
      , app = express()
      , index = require('./routes/index'); 

    app.use('/rest', index); 
    // etc
```

`app.user` indicate to express that all route which begin with `/rest` will be redirected to routes which are in the `routes/index.js` file. 


{% img /images/curl_from_router.png  %}

### Helpers

You notice, that each time we made a modification to any __NodeJS__ file, we must rerun the server, which is not very handy. 

Fortunatly a npm package, called `nodemon` is here to assure hot reload code for NodeJS files. 

```bash install nodemon
$ npm install -g nodemon # Install the package
$ nodemon server.js # Run the server
```

{% img /images/nodemon.png  %}


Another intersting feature would be the ability to have a log feature, that will simply display an output into the node.js console each time a route has been called. 


```javascript server.js
// The logging function
function log_node(callback){
    return function(req, res, next) {
        console.log(new Date().toDateString() + " : " + req.method + " => " + req.url);
        callback.call(null, req, res, next);
    }
}
```

Apply the function to the use method. 

```javascript server.js
// Change the line and add the function
app.use('/', log_node(index));

```
Query the route and you will see an output in the console ! 

This function, is very simple and you can obviously create a specific module, with some filter for particular route, colors, etc. 

## Mongoose + NodeJS

Now that we isolate the business layer, it's time to use the persistence layer.
I present you earlier __mongoose__ which provides a straight-forward, schema-based solution to modeling your application data and includes built-in type casting, validation, query building, business logic hooks and more, out of the box.

```javascript example of use
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var Cat = mongoose.model('Cat', { name: String });

var kitty = new Cat({ name: 'Zildjian' });
kitty.save(function (err) {
  if (err) // ...
  console.log('meow');
});
```

To start with mongoose, let's create first a configuration file in a folder called __conf__. 


```javascript back/conf/general.js

module.exports = (function(){

    exports.mongodb = {
        host: 'localhost',
        db: 'mean',
        opt: {}
    };

    return exports;
})();

```

Add the configuration file to the server. 

```javascript server.js

app = express(),
conf = require('./conf/general'), // Our configuration
mongoose = require('mongoose');  // Mongoose library

// ...

```

Now create some connection function, in order to connect to mongo. 

```javascript server.js
// Connection
mongoose.connect('mongodb://'+ conf.mongodb.host + '/' + conf.mongodb.db, conf.mongodb.opt);

// Assign event to the MongoDB instance.
mongoose.connection.on('open', function(ref){
    console.log("connection to mongo server " + conf.mongodb.host);
});

mongoose.connection.on('error', function(err){
    console.log(err);
});

mongoose.connection.on('disconnected', function(){
    console.log("Disconnected");
});
```

Cool we can now connect our application to the database. 

__Remeber__ to start the MongoDb server with `mongod` before, launching the server or a beautiful error will show up ;) !

## Data models

Data in MongoDB has a flexible schema. Collections do not enforce document structure. This flexibility gives you data-modeling choices to match your application and its performance requirements.

In mongoDB, the data models are representend by Schema (which are simply JSON object). 

[More about MongoDb's data modeling](http://docs.mongodb.org/manual/data-modeling/)

Mongoose, allow us to easily design our pattern of data. 
Create one called fooBar

```javascript models/fooBar.js
var mongoose = require('mongoose')
  , Schema = mongoose.Schema;

/**
* fooBar model. 
* name : String
* type : String 
* toString() : String  
* 
* A really nice model. 
*/
'use strict';

exports.schema = new Schema({
    name: String,
    type: String
});

exports.schema.statics.toString = function(){
    return "Hello i'm " + this.name + " a  " + this.type + " schema ! ";
};

// Very important, in order to compile the mongoose's schema definition.
module.exports = mongoose.model('fooBar', exports.schema);
```

Let's use this schema in our controller. 

```javascript routes/index.js
var fooBar = require('../models/fooBar');

...

router.get('/fooBar', function(req, res){
    new fooBar({
        name: "Louis",
        type: "JavaScript loverz"
    }).save(function(err){
        if(err) throw err;
        res.send(" It's in the databases now ! ");
    });
});
```

I should have normaly use a POST method, in order to persist the new document, but for the purpose of the example, i use a GET. 


{% img /images/fooBarnew.png  %}

Open a new terminal and go the mongo cli to verify that our model has beed saved. 

{% img /images/mongoConsole.png  %}

Cool, isn't it ? :) 

# Front-end part. 

It's not an obligation to use that tool, and you can grab all the angular component by hand and generate your own directory structure. 

```bash 
$ cd front && yo angular
``` 

This command will, create the basic architecture of the angular application.

There are small questions in order to choose usefull library as _angular-route_ or _angular-animate_.

Select __angular-route__.

Yeoman will fetch and install all the library you need with _npm install_ and _bower install_. 

```bash the generated directory structure
.
├── Gruntfile.js
├── app/
│   ├── 404.html
│   ├── favicon.ico
│   ├── images/
│   │   └── yeoman.png
│   ├── index.html
│   ├── robots.txt
│   ├── scripts/
│   │   ├── app.js
│   │   ├── controllers
│   │   │   ├── about.js
│   │   │   ├── main.js
│   ├── styles/
│   │   └── main.css
│   └── views/
│       ├── about.html
│       ├── main.html
├── bower.json
├── bower_components/ # Bower library
└── node_modules/ # Node library use for grunt
├── package.json 
└── test/ # Karma test folder 
```


The angular-generator comes with a task manager called [Grunt](http://gruntjs.com/)  and a package manager called [Bower](http://bower.io/).

- __Grunt__ will be very usefull to launch task as minification, test or hot-reload. 
- __Bower__ will be used to fetch and install packages from all over, taking care of hunting, finding, downloading, and saving the stuff you’re looking for. Bower keeps track of these packages in a manifest file, bower.json.


## Begin with yeoman

### Create a simple route 

Yeoman as a good generator, will allow us to save time and will create very quickly all file required.

For example let create our first route. 

```bash 
$ yo angular:route foo
```

Yeoman going to automaticaly add a route in `app.js`

```javascript app.js
.when('/foo', {
    templateUrl: 'views/foo.html',
    controller: 'FooCtrl'
})
```

To access the page, we have two solutions. 

+ Access the front folder, with a Apache or NGiNX server.
+ Use the grunt task `server` in order to emulate a server. 

It will function similar to an Apache server, serving up static files based on their path, but uses the http module via connect to set it up.

In addition to run the server, this task will passing all Karma test, activate hot-reload by watching file for modification. 

All the task are register in the __Gruntfile.js__ file. 


```bash front/
$ grunt serve
```

Browse the `http://localhost:9000/#/foo` et voila ! :)

Go to the part [two]({{ root_url }}/blog/2014/10/02/hacking-mean-part2/).  

For any questions or additional information, please feel free to comment ! 

# Troubleshooting 

## Fatal error: Unable to find local grunt. (Front)

Grunt does not install correctly. 

```bash 
$ sudo npm install grunt --save-dev
$ npm install 
```



## ERROR: dbpath (/data/db) does not exist. (Mongod)

MongoDb need a path to store all the data. 

```bash
$ sudo mongod --dbpath=/path/to/your/data/folder
```



