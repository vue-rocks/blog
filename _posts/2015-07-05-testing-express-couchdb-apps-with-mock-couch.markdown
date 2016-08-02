---
layout: post
title:  "Testing Express.js + CouchDB Applications with mock-couch"
description: "There's a way to test Express.js + CouchDB web apps with confidence of end-to-end tests and execution speed of unit-tests. Meet mock-couch- a node.js module which pretends to be a CouchDB server for testing purposes."
date:   2015-07-05 22:30:00+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: justas
---

## Prerequisites

The code in this post is based upon [CoffeeScript][coffee], [Mocha][mocha] and [Gulp][gulp] but it should be easily adaptable to other stacks (e.g. [JavaScript][js], [Jasmine][jasmine] and [Grunt][grunt]).

## Intro

Recently, I worked on a [Express.js][express] based RESTful API with [CouchDB][couch] storage. For various reasons, I mostly had end-to-end tests (testing HTTP endpoints) and very few unit tests. In the tests, I would rebuild the database before every `describe` block. This worked quite well but at one point I noticed that my feedback loops were beginning to get tediously slow (~30 seconds to run the full test suite). Thankfully, I discovered [mock-couch][mock-couch]: in-memory storage with CouchDB API. The switch was rather easy and, as a result, my test suite would finish in less than 10 seconds (even when reseting test data before each test).

## Setup

Our sample API will expose functionality to access a database of sofa's. Let's set it up.

Structure and dependencies


```bash
mkdir -p sofa-api/specs
cd sofa-api
npm install mock-couch coffee-script gulp gulp-mocha should supertest express nano q
```

gulpfile.coffee


```coffeescript
require 'coffee-script/register'

gulp    = require 'gulp'
mocha   = require 'gulp-mocha'

sources =
  tests: './specs/**/*.spec.coffee'

# Task to run tests with mock-couch backend
gulp.task 'test', ->
  process.env.NODE_ENV = 'test'
  process.env.PORT = 3010
  process.env.COUCH_PORT = 5987
  process.env.DATABASE = 'sofas_test'
  gulp.src(sources.tests)
    .pipe(mocha())

# Task to run integration tests (real CouchDB backend)
gulp.task 'test-int', ->
  process.env.NODE_ENV = 'test-int'
  process.env.PORT = 3010
  process.env.COUCH_PORT = 5984
  process.env.DATABASE = 'sofas_test'
  gulp.src(sources.tests)
    .pipe(mocha())

# Task to run our server
gulp.task 'default', ->
  process.env.NODE_ENV = 'development'
  process.env.PORT = 3000
  process.env.COUCH_PORT = 5984
  process.env.DATABASE = 'sofas'

  server = require './server'
  server.listen()
```

server.coffee


```coffeescript
express    = require 'express'

app = express()

app.get '/sofas', (req, res) ->
  throw new Error 'not implemented'

app.get '/sofas/:sofa', (req, res) ->
  throw new Error 'not implemented'


exports.listen = (callback) ->
  port = process.env.PORT
  app.listen port, callback
```

Let's create file *specs/sofas.spec.coffee* which is going to contain our tests.


```coffeescript
should = require 'should'

describe 'Sofas API', ->

  describe '/sofas', ->
    it 'should return all sofas with GET', ->
      # TODO: write test
      (false).should.be.true()

  describe '/sofas/:sofa', ->
    it 'should return a particular sofa with GET', ->
      # TODO: write test
      (false).should.be.true()
```

If you run the tests right now, you should see similar output:


```bash
gulp test

  Sofas API
    /sofas
       1) should return all sofas with GET
    /sofas/:sofa
       2) should return a particular sofa with GET


  0 passing (30ms)
  2 failing

  1) Sofas API /sofas should return all sofas with GET:
     AssertionError: expected false to be true
    at Context.<anonymous> (specs/sofas.spec.coffee:8:7)
  

  2) Sofas API /sofas/:sofa should return a particular sofa with GET:
     AssertionError: expected false to be true
    at Context.<anonymous> (specs/sofas.spec.coffee:13:7)
  


Message:
    2 tests failed.
```

## Implementation

Sofas will be stored in CouchDB with the following structure:


```json
{
    "_id": "CouchDB assigned id",
    "_rev": "CouchDB assigned revision",
    "name": "Name of the sofa",
    "price": "Price of the sofa"
}
```

With this in mind, let's build a module for accessing the database.


```coffeescript
nano = require 'nano'
Q    = require 'q'

class DAL
  constructor: (port, @database) ->
    @conn = nano("http://localhost:#{port}")

  # Returns a promise reseting database
  resetDatabase: (withSamples=false) ->
    docs = [@_sofaViews]
    if withSamples
      docs = docs.concat @_sofaSamples

    @_deleteDb()
      .then => @_createDb()
      .then => Q.all(docs.map (d) => @_insertDoc(d))

  # Returns a promise of sofas
  # If you provide the id- you'll get an array with a particular sofa in it
  getSofas: (id=null) ->
    params = {}
    params.key = id if id?

    @_view('sofas', 'all', params)
      .then (res) ->
        body = res[0]
        sofas = (row.value for row in body.rows)
        return sofas

  _sofaViews:
    _id: '_design/sofas'
    views:
      all:
        map: (doc) ->
          if doc.type is 'Sofa'
            emit doc._id, doc

  _sofaSamples: [
    {
      _id: 'sofa1'
      type: 'Sofa'
      name: 'Sofa1'
      price: 400
    }
    {
      _id: 'sofa2'
      type: 'Sofa'
      name: 'Sofa2'
      price: 300
    }
    {
      _id: 'sofa3'
      type: 'Sofa'
      name: 'Sofa3'
      price: 500
    }
  ]

  # Returns promise of creating database
  _createDb: ->
    d = Q.defer()
    @conn.db.create @database, d.makeNodeResolver()
    d.promise

  # Returns promise of deleting database
  _deleteDb: (ignoreMissing=true)->
    d = Q.defer()
    @conn.db.destroy @database, (err, res) ->
      err = null if err? and err.statusCode is 404 and ignoreMissing
      d.makeNodeResolver()(err, res)
    d.promise


  # Returns promise of inserting a doc
  _insertDoc: (doc) ->
    d = Q.defer()
    @conn.use(@database).insert doc, d.makeNodeResolver()
    d.promise


  # Returns promise of CouchDB view
  _view: (designDoc, view, params={}) ->
    d = Q.defer()
    @conn.use(@database).view designDoc, view, params, d.makeNodeResolver()
    d.promise


module.exports = DAL
```

What's missing is the endpoint and test implementation. Let's start with the tests.


```bash
diff --git a/specs/sofas.spec.coffee b/specs/sofas.spec.coffee
index 8c1eb84..efbff58 100644
--- a/specs/sofas.spec.coffee
+++ b/specs/sofas.spec.coffee
@@ -1,13 +1,54 @@
-should = require 'should'
+should  = require 'should'
+DAL     = require '../dal'
+request = require 'supertest'
+request = request "http://localhost:#{process.env.PORT}"
+server  = require '../server'
+
+app = null
 
 describe 'Sofas API', ->
 
+  beforeEach (done) ->
+    # Reset database and start express server before each test
+    dal = new DAL(process.env.COUCH_PORT, process.env.DATABASE)
+    dal.resetDatabase(true)
+      .then ->
+        app = server.listen done
+      .fail (err) -> done err
+
+  # Close express server after each test
+  afterEach (done) ->
+    app.close done
+
+
   describe '/sofas', ->
-    it 'should return all sofas with GET', ->
-      # TODO: write test
-      (false).should.be.true()
+    it 'should return all sofas with GET', (done) ->
+      request
+        .get '/sofas'
+        .expect 200
+        .expect (res) ->
+          sofas = res.body
+          sofas.should.have.length(3)
+          for s in sofas
+            s.should.have.property('_id')
+            s.should.have.property('_rev')
+            s.should.have.property('name')
+            s.should.have.property('price')
+
+        .end done
+          
+
 
   describe '/sofas/:sofa', ->
-    it 'should return a particular sofa with GET', ->
-      # TODO: write test
-      (false).should.be.true()
+    it 'should return a particular sofa with GET', (done) ->
+      request
+        .get '/sofas/sofa1'
+        .expect 200
+        .expect (res) ->
+          sofa = res.body
+          sofa.should.have.property('_id')
+          sofa.should.have.property('_rev')
+          sofa.should.have.property('name')
+          sofa.should.have.property('price')
+
+        .end done
```

Now the endpoints:


```bash
diff --git a/server.coffee b/server.coffee
index 98f1918..a0d30ce 100644
--- a/server.coffee
+++ b/server.coffee
@@ -1,16 +1,27 @@
 express    = require 'express'
+DAL        = require './dal'
 
+dal = new DAL(process.env.COUCH_PORT, process.env.DATABASE)
 app = express()
 
 app.get '/sofas', (req, res) ->
-  throw new Error 'not implemented'
+  dal.getSofas()
+    .then (sofas) ->
+      res.json(sofas)
+    .fail (err) -> res.status(err.statusCode).send()
 
 app.get '/sofas/:sofa', (req, res) ->
-  throw new Error 'not implemented'
+  dal.getSofas(req.params.sofa)
+    .then (sofas) ->
+      if sofas.length isnt 1
+        res.status(404).send()
+      else
+        res.json(sofas[0])
+    .fail (err) -> res.status(err.statusCode).send()
 
 
 exports.listen = (callback) ->
```

At this point we can execute <em>gulp test-int</em> command to run tests on real CouchDB:


```bash
gulp test-int

  Sofas API
    /sofas
       ✓ should return all sofas with GET
    /sofas/:sofa
       ✓ should return a particular sofa with GET


  2 passing (204ms)
```

## mock-couch tests

So how do we use mock-couch for this? Mock-couch is an HTTP server based on [Restify][restify]. Note that [Node.js][node] runtime is single-threaded therefore we won't be able to run both express and mock-couch at the same time. However, it is possible to run mock-couch on another process.

Node provides a couple of ways to create processes out of the box. We'll use one called [forking][fork]: not only we'll be able to control the lifecycle of the forked child process but we'll also have the ability to send and receive messages between parent and the child.

OK, less talk and more forking. We'll do the forking in <em>gulp test</em> task:


```bash
diff --git a/gulpfile.coffee b/gulpfile.coffee
index cbd61b5..1200834 100644
--- a/gulpfile.coffee
+++ b/gulpfile.coffee
@@ -6,14 +6,48 @@ mocha   = require 'gulp-mocha'
 sources =
   tests: './specs/**/*.spec.coffee'
 
+
 # Task to run tests with mock-couch backend
 gulp.task 'test', ->
   process.env.NODE_ENV = 'test'
   process.env.PORT = 3010
   process.env.COUCH_PORT = 5987
   process.env.DATABASE = 'sofas_test'
-  gulp.src(sources.tests)
-    .pipe(mocha())
+
+  fork = require('child_process').fork
+  couchProcess = fork('node_modules/gulp/bin/gulp.js', ['mock-couch'], {cwd: __dirname})
+  couchProcess.on 'message', (msg) =>
+    if msg is 'listening'
+      gulp.src(sources.tests)
+        .pipe(mocha())
+        .once('error', (err) ->
+          process.exit(1)
+        )
+        .once('end', ->
+          process.exit()
+        )
+
+  process.on 'exit', ->
+    couchProcess.kill()
+
+gulp.task 'mock-couch', ->
+  process.env.NODE_ENV = 'test'
+  process.env.PORT = 3010
+  process.env.COUCH_PORT = 5987
+  process.env.DATABASE = 'sofas_test'
+
+  mockCouch = require 'mock-couch'
+  couchdb = mockCouch.createServer()
+  DAL = require './dal'
+  dal = new DAL(process.env.COUCH_PORT, process.env.DATABASE)
+  docs = [dal._sofaViews]
+  docs = docs.concat dal._sofaSamples
+
+  couchdb.addDB process.env.DATABASE, docs
+
+  couchdb.listen process.env.COUCH_PORT, ->
+    process.send 'listening'
+
 
 # Task to run integration tests (real CouchDB backend)
 gulp.task 'test-int', ->
```

Also, we need a slight change to the test setup:


```bash
diff --git a/specs/sofas.spec.coffee b/specs/sofas.spec.coffee
index efbff58..93e7bd2 100644
--- a/specs/sofas.spec.coffee
+++ b/specs/sofas.spec.coffee
@@ -9,12 +9,17 @@ app = null
 describe 'Sofas API', ->
 
   beforeEach (done) ->
-    # Reset database and start express server before each test
-    dal = new DAL(process.env.COUCH_PORT, process.env.DATABASE)
-    dal.resetDatabase(true)
-      .then ->
-        app = server.listen done
-      .fail (err) -> done err
+
+    if process.env.NODE_ENV is 'test-int'
+      # Reset database and start express server before each test
+      dal = new DAL(process.env.COUCH_PORT, process.env.DATABASE)
+      dal.resetDatabase(true)
+        .then ->
+          app = server.listen done
+        .fail (err) -> done err
+    else
+      app = server.listen done
+
 
   # Close express server after each test
   afterEach (done) ->
```

## Moment of Truth

If you try running `gulp test` right now, you should see similar output:


```bash
gulp test

  Sofas API
    /sofas
       ✓ should return all sofas with GET (125ms)
    /sofas/:sofa
       ✓ should return a particular sofa with GET (41ms)


  2 passing (178ms)

```

As you can see the tests completed slightly faster than with real CouchDB. In a small code base like this, it's hardly worth switching to mock-couch, if execution speed is your only motivation. However, with bigger codebases, the speed improvement can become very apparent.

Besides speed, there's another benefit to this approach: you don't actually need CouchDB to test your project. The simplified build process is especially useful with Continuous Integration platforms like [Travis CI][travis] and [Codeship][codeship].

## Resources

- [Code on GitHub][showcase]
- [mock-couch][mock-couch]


[coffee]: http://coffeescript.org/
[mocha]: http://mochajs.org/
[gulp]: http://gulpjs.com/
[js]: https://www.javascript.com/
[jasmine]: https://jasmine.github.io/
[grunt]: http://gruntjs.com/
[express]: http://expressjs.com/
[couch]: https://couchdb.apache.org/
[mock-couch]: https://github.com/chris-l/mock-couch
[restify]: http://mcavage.me/node-restify/
[node]: https://nodejs.org/
[fork]: https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options
[travis]: http://travis-ci.org/
[codeship]: https://codeship.com
[showcase]: https://github.com/reederz/express-mock-couch-showcase
