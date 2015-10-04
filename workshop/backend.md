[ Index ](../README.md)

# Backend

## Node.js

* JavaScript on the server
  * In theory can be written in any language which can compile to JavaScript, including [CoffeeScript](http://coffeescript.org), [TypeScript](http://typescriptlang.org), [Dart](http://dartlang.org), [ClojureScript](https://github.com/clojure/clojurescript) etc. 
* Strictly speaking, Node is a _runtime_: a set of low-level routines written in C++ based around Chrome's V8 engine 
  * V8 is a compiler, not an interpreter: implications for speed and resource consumption
* Unlike browser-based JavaScript, Node has access to server resources including files on disk, networking, and other features of the operating system
* Node's package manager, [npm](https://www.npmjs.com), has become incredibly popular
* Dates back to 2009, with meteoric rise in popularity since then


### Non-blocking I/O

* Part of Node's appeal is its ability to execute _asynchronous_ code
* Node operates on a single thread, but all functions which perform input/output operate from a separate pool of threads managed by [libuv](https://github.com/libuv/libuv)
  * Node waits for a callback but can do other things while it waits. This behaviour is said to be _event-driven_ (sometimes called the _event loop_).
* Design allows thousands of concurrent requests
* Can't make use of multi-core processors without explicitly handling child processes (for example, using the `cluster` module)


### `require` vs `import`

* Node's module system came from [CommonJS](https://en.wikipedia.org/wiki/CommonJS). You may encounter a lot of code that looks like this:

    ```js
    var http = require('http');
    var foo = require('foo');
    ```

* It's certainly possible to use _ES6_ `import`s:

    ```js
    import http from 'http';
    import path from 'path';
    ```

* Because these features are fairly new to the language, it's necessary to use a _transpiler_ to translate the ES6 features of the language into something that can be readily understood by Node. As the toolset matures, this should gradually become unnecessary.

    ```js
    var gulp = require('gulp');
    var babel = require('gulp-babel');

    gulp.task('default', function() {
        return gulp.src('server.js')
            .pipe(babel())
            .pipe(gulp.dest('dist'));
    });
    ```

  * Here we use [Babel](https://babeljs.io) to process the server file, then output to the `dist` directory


### Further Reading

* [Understanding the node.js event loop](http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/)


## Express

* [Express](http://expressjs.com) is a minimalist web framework that sits on top of Node.
* Has features for defining models, serving static content, and exposing APIs
* For example, a minimal HTTP server can be written like so:

    ```js
    import http from 'http';
    import path from 'path';
    import express from 'express';

    let router = express();
    let server = http.createServer(router);

    router.use(express.static(path.resolve(__dirname, '../client')));

    server.listen(process.env.PORT || 3000, process.env.IP || "0.0.0.0", function(){
      console.log("Ok");
    });
    ```

  * This serves everything in the `client` directory as static content
  * The port to listen on is held in the environment variable `$PORT`


### Everything is 'middleware'

* In Express, almost every part of request handling is treated as _middleware_.
  * Flow of execution is passed from middleware to middleware using the `next` function.

    ```js
    // app.js
    import authenticate from 'authenticate';
    import aardvarks from 'aardvarks';
 
    app.use('/aardvarks', authenticate);
    app.use('/aardvarks', aardvarks);


    // authenticate.js
    app.all('/aardvarks', function (req, res, next) {
      checkUserAuth().then(function (user) {
        if (user.isAuthenticated()) {
          req.user = user;
          next();
        }
        res.status(401).json({ message: 'Please login to access this resource.' });
      });
    });


    // aardvarks.js
    app.get('/aardvarks', function (req, res, next) {
      let aardvark = new Aardvark();
      res.json(aardvark);
    });
    ```

  * This provides security because the order of execution is guaranteed: the `authenticate` middleware will always execute before `aardvarks`
    * If the user fails authentication, a response is written with HTTP status code 401
  * Notice that before the call to next, the request object is modified directly with new information (the user object)
    * Careful: it's possible to accidentally overwrite properties on the request
  * Middleware can be built up in this fashion into a _middleware stack_


## Feathers

* [Feathers](http://feathersjs.com) is an [Express](#express) wrapper that takes care of boilerplate tasks
* Makes setting up a REST API fairly trivial:

    ```js
    import feathers from 'feathers';
    import bodyParser from 'body-parser';
    import mongoService from 'feathers-mongo';

    let app = feathers()
      .configure(feathers.rest());
      .use(bodyParser.json())
      .use('aardvarks', new mongoService('aardvarks'));

    let port = 8080;
    app.listen(port, function () {
      console.log(`Listening for Orycteropus afer on port ${port}...`);
    });
    ```

[ Index ](../README.md)
