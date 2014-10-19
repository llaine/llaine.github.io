---
layout: post
title: "hacking MEAN get started with your own, part 2"
date: 2014-10-02 08:51:35 +0200
comments: true
categories: JavaScript NodeJS AngularJS MongoDb NoSQL
description: "Hacking MEAN stack."
keywords: "miage, bordeaux, louis lainé, lainé, louis, javascript, nodejs, github, angularjs, ecmascript6"
---


Now that we have both part operationnal, let's create a simple application. 

I'm a music lover, so why don't create a simple music library. :)



# Models


```javascript song.js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;
/**
* The song model 
* @name : String, the name
* @author: String the author
* @releaseDate : String, the release date of the song 
* @styles: Array of string, represent all the style a song may have. 
*/

'use strict';

exports.schema = new Schema({
    name: String,
    author: String,
    releaseDate: String,
    styles: [String]
});

module.exports =  mongoose.model('music', exports.schema);



```

```javascript library.js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;
/**
* The library model 
* @name : String, the name
* @creationDate : Date.now
* @releaseDate : String, the release date of the song 
* @styles: Array of string, represent all the style a song may have. 
*/
'use strict';

exports.schema = new Schema({
    name: String,
    creationDate: { type: Date, default: Date.now },
    music: [Schema.Types.ObjectId]
});

model.exports = mongoose.model('library', exports.schema);
```

We will create a simple CRUD for the library and for the song. 

# Back end 


## Library models

What we are going to create, is a RESTFul API. 
Using express and mongoose functions to save/find and send our datas. 

I create CRUD functions, which are going to be called by my REST Service

For the application, i create a simple helpers, which is going to help us handle the mongoDb response. 

```javascript helpers/utils.js
// ...
/**
 * Handle a MongoDb response
 * @param err
 * @param r
 * @returns {*}
 */
helper.handleResponse = function(err, r){
    try {
        if(err) throw err;
    } catch(e){
        helper.log(e.message);
    }
    return r;
};
```
Then in my library service.
```javascript libraryService.js


 /**
 * GET -> /rest/library/
 */
router.get('/', function(req, res) {

    /* find all document in the library collection */
    libraryRepo.find().exec(function(err, r){

        /* handle response */
        res.send(
            helpers.handleResponse(err, r)
        );

    });

});

/**
 * GET -> /rest/library/:id
 */
router.get('/:id', function(req, res){

    /* _id ObjectId type. */
    libraryRepo.findOne(
        {
            _id: new mongoose.Types.ObjectId(req.params.id)
        }
    ).exec(function(err, r){

        /* handle response */
        res.send(
            helpers.handleResponse(err, r)
        );

    });
});

/**
 * POST -> /rest/library
 */
router.post('/', function(req, res){

    /* create a new library document */
    new libraryRepo({
        name: req.body.name,
        music: req.body.songs
    })
    .save(function(err){

        /* send a 200 if everything ok ! */
        res.status(
            helpers.handleResponse(err, 200)
        ).end();

    });


});

```

Mongoose is very functional and uses the same query language as the MongoDb console. Very helpful to train. 

I also use `exec()` in all my mongo call. 
This function, allow you to chain database queries and execute them at the end with all the predicate you selected. 

[Mongoose Query Language](http://mongoosejs.com/docs/populate.html)

You also notice that i used __ObjectId__ to find by id.
ObjectId is a 12-byte BSON type, constructed using:

- a 4-byte value representing the seconds since the Unix epoch,
- a 3-byte machine identifier,
- a 2-byte process id, and
- a 3-byte counter, starting with a random value.

In MongoDB, documents stored in a collection require a unique _id field that acts as a primary key. Because ObjectIds are small, most likely unique, and fast to generate, MongoDB uses ObjectIds as the default value for the _id field if the _id field is not specified. 

### Testing the API

To test the service, i used [PostMAN](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm)
which is REST client for testing API. 
You can use old good `curl -X POST`, but Postman is very usefull. 

So after populating my database with some documents, i could get all resources.. 

```json /rest/library -X GET
[
    {
        "_id": "542d37f66e2a40ee33bd6491",
        "name": "test",
        "__v": 0,
        "music": [],
        "creationDate": "2014-10-02T11:33:10.501Z"
    },
    ...
]
```

I can also easily access to a particular resource.

```json /rest/library/542d37e3856d759d33ba3d40
{
    "_id": "542d37e3856d759d33ba3d40",
    "name": "test",
    "__v": 0,
    "music": [],
    "creationDate": "2014-10-02T11:32:51.985Z"
}
```

## Song models 

Pretty much the same logic here. 

```javascript songService.js
/**
 * GET -> /rest/song/
 */
router.get('/', function(req, res){

    /* fetch all the song */
    songRepo.find().exec(function(err, r){

        /* handle response */
        res.send(
            helpers.handleResponse(err, r)
        );

    });

});

/**
 * GET -> /rest/song/:id
 */
router.get('/:id', function(req, res){

    /* _id ObjectId type. */
    songRepo.findOne(
        {
            _id: new mongoose.Types.ObjectId(req.params.id)
        }
    ).exec(function(err, r){

        /* handle response */
        res.send(
            helpers.handleResponse(err, r)
        );

    });

});

/**
 * POST -> /rest/song
 * Create a new song.
 */
router.post('/', function(req, res){
    /* create a document from the POST params */
    new songRepo({
        name: req.body.name,
        author: req.body.author,
        releaseDate: req.body.releaseDate,
        styles: req.body.styles
    }).
    save(function(err){

        /* send a 200 code if everything's ok */
        res.send(
            helpers.handleResponse(err, 200)
        );

    });

});
```

The same thing as before when it's about testing the new routes. 
```json /rest/song -X GET
[
    {
        "_id": "542d525dc90c5f4f50642493",
        "name": "Giant Steps",
        "author": "John Coltrane",
        "releaseDate": "January 1960",
        "__v": 0,
        "styles": [
            ""
        ]
    },
    ...
]
```

```json /rest/song/542d525dc90c5f4f50642493
{
    "_id": "542d525dc90c5f4f50642493",
    "name": "Giant Steps",
    "author": "John Coltrane",
    "releaseDate": "1970-01-01",
    "__v": 0,
    "styles": [
        ""
    ]
}
```



# Front end 

Now that all our back-end part is ready let us deal with the front :) ! 

## Services 

Create a simple service, which allow us to get resource from the backend.

```bash 
$ yo angular:service libraryService
```
We are going to get all resources in order to acces the lbrary resource. 

```javascript  services/libraryService.js
/**
 * @ngdoc service
 * @name frontApp.libraryService
 * @description
 * # libraryService
 * Service in the frontApp.
 */
angular.module('frontAppServices', [])
  .factory('libraryService', ['$http', 'BASE_URL', '$q', function($http, BASE_URL) {

        return {
            /**
             * GET -> Get all the libraries
             * @returns {HttpPromise}
             */
            get: function(){
                return $http.get(BASE_URL + '/rest/library');
            },
            /**
             * GET -> Get one library
             * @param id
             * @returns {HttpPromise}
             */
            getById: function(id){
                return $http.get(BASE_URL + '/rest/library/get/'+ id);
            },
            /**
             * POST -> Create a new library
             * @param libInfos
             */
            createNew: function(libInfos){
               console.log(libInfos);
            },
            /**
             * PUT -> Add new song in the current library music array.
             * @param idLibrary
             * @param theNewSongArray
             * @returns {HttpPromise}
             */
            addSong: function(idLibrary, theNewSongArray){
                return $http({
                    method: 'PUT',
                    url: BASE_URL + '/rest/library/update/' + idLibrary,
                    data: {
                        songs:theNewSongArray
                    }
                });
            }

        };
  }]);
```

Don't forget to add this module in the main module or angular won't be able to load it.


```javascript app.js
angular
  .module('frontApp', [
    'ngRoute',
    'frontAppServices'
  ])
  .config(function ($routeProvider) {
  //...
```

Now i need to call my service in the main controller. 

```javascript controllers/main.js
libraryService.get().then(function(data){

    if(data.status === 200){
        /* access to the data informations */
        $scope.theLibraries = data.data;
    }
});
```

Now i want to access all music from a single library. 

```bash 
$ yo angular:route library
```

Use the libraryService in the controller. 

For example : 

```javascript library.js


    /* the current library id */
    var libraryId = $routeParams.id
      , newSongToAdd = [];

    /**
     * Handle $http promise
     * @param response
     * @returns {*}
     */
    function checkForResponse(response){
        if(response.status === 200){
            return response.data;
        }
    }

    /**
     * Add or remove the item from the array
     * @param item
     */
    function addToArray(item){
        var position = newSongToAdd.indexOf(item);
        /* when > -1 mean dat is present */
        if(position === -1){
            newSongToAdd.push(item);
        }else{
            newSongToAdd.splice(position, 1);
        }
    }

    /**
     * add the selected song to the song array.
     * @param idSong
     */
    $scope.addSong = function(idSong){
        addToArray(idSong);
    };

    /**
     *  Validation the new array and save him in the database.
     */
    $scope.validNewSongs = function(){
        /* let's assume that the array isn't empty */
        if(newSongToAdd.length > 0){

            libraryService.addSong(newSongToAdd).then(function(data){
                console.log(data);
            });

        }
    };

    /* call to libraryService which retrive all resources */
    libraryService.getById(libraryId).then(function(data){
        /* store the library information */
        var informationsLibrary = checkForResponse(data);
        /* display them in the partials */
        $scope.currentLibrary = informationsLibrary;

        $scope.isMusic = informationsLibrary.musics === undefined;

    });

    /* call to the songService which retrieve all resources */
    songService.get().then(function(data){
        /* display all song in the partials. */
        $scope.songs = checkForResponse(data);

    });
```

You can now manipulate all resources.


# Conclusion 


In conclusion, we saw how to create a communication between the two layer by following good manners. 

Feel free to comments, for any information :) 

You can of course, find all the code we seen in [github](https://github.com/llaine/blog-hackingMean)








