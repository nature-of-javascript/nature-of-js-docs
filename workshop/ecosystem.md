[ Index ](../README.md)

# Ecosystem

Today, the JavaScript ecosystem is in a constant state of ... innovation ... as millions of developers contribute to client-side and server-side libraries and tools.

We couldn't possibly present an exhaustive overview of the ecosystem, but we think the following tools are mandatory for most JavaScript developers to at least be familiar with.

## npm

* Since its release in 2009, the [Node package manager](https://www.npmjs.com) has become one of the most prolific package managers in the world 
* Share packaged modules of JavaScript code
* Stipulate versions (or version ranges) for package dependencies
* npm Registry contains over 195,000 modules: the largest of all package repositories 
* Can install packages _locally_ or _globally_
* Controlled by `package.json`:

    ```js
    {
      "name": "Aardvarks Ahoy",
      "version": "0.0.0",
      "description": "Orycterapus after",
      "main": "server.js",
      "repository": "https://iamnotarealrepository",
      "author": "Rich Churcher <foo@wombat.com>",
      "dependencies": {
        "express": "~3.2.4",
      },
      "devDependencies": {
        "babel": "^5.8.23",
        "babelify": "^6.3.0",
        "browserify": "^11.1.0",
        "gulp": "^3.9.0",
        "gulp-babel": "^5.2.1",
        "gulp-concat": "^2.6.0",
        "gulp-uglify": "^1.4.1",
        "karma": "^0.13.9",
        "minifyify": "^7.0.6",
        "vinyl-source-stream": "^1.1.0"
      }
    }
    ```

  * Each package keeps a copy of its own dependencies in a nested folder structure (under `node_modules`)
    * This causes a lot of duplication on the server's file system, but since we're not running JavaScript in the browser it's rarely a problem


## Bower

* Bower is ideally suited to managing packages for the client, because unlike npm it doesn't nest dependencies: there's only one copy of each
  * Much lighter weight, faster page loads
  * Interacts with Git, can install directly from a repository
  * Very common to use a combination of Bower and npm: they're not mutually exclusive

* Controlled by bower.json:

    ```js
    {
      "name": "Aardvarks Ahoy",
      "description": "Orycterapus after",
      "version": "0.0.0",
      "homepage": "https://thisisnotthehomepage.io",
      "authors": [
        "Rich Churcher <foo@wombat.com>"
      ],
      "main": "js/app.js",
      "moduleType": [
        "amd",
        "es6",
        "node"
      ],
      "keywords": [
        "aardvark",
        "landlubber",
        "keelhaul"
      ],
      "license": "MIT",
      "ignore": [
        "**/.*",
        "node_modules",
        "bower_components",
        "test",
        "tests"
      ],
      "dependencies": {
        "jquery": "~2.1.4",
        "lodash": "~3.10.1"
      }
    }
    ```

* The above or something very like it would be generated with the following terminal commands:

    ```bash
    npm i bower --save-dev
    bower init
    bower install jquery lodash --save
    ```

  * `bower init` fills most of the details out for you, including grabbing the GitHub repo as homepage, and adding any previously installed components
  * Subsequently, components can be included directly from `bower_components/packagename/packagename.js` or similar


## Gulp

* Unlike [npm](#npm) and [Bower](#bower), Gulp is a _build system_ and not a package manager
* Also referred to as a _task runner_.
* Typically used for:
  * Concatenation and minification of source files, other deployment-related tasks
  * Compiling your CSS/Sass
  * Running a transpiler, allowing you to use _ES6_ features
  * Running test suites
  * Compressing images
  * Watching files for changes
* Gulp can be used with both client and server-side projects
* Uses node streams
  * Reads source files, pipes the text through plugins, and writes output
* Wide variety of plugins: uses [npm](#npm) packages
* Compares to [Grunt](http://gruntjs.com), another widely-used task runner

* Controlled with `gulpfile.js`. For example:

    ```js
    var gulp = require('gulp');
    var concat = require('gulp-concat');
    var stripDebug = require('gulp-strip-debug');
    var uglify = require('gulp-uglify');
    var sourcemaps = require('gulp-sourcemaps');
    var Server = require('karma').Server;
     
    gulp.task('js', function () {
      return gulp.src('js/**/*.js')
        .pipe(sourcemaps.init())
          .pipe(concat('app.js'))
          .pipe(stripDebug())
          .pipe(uglify(true))
        .pipe(sourcemaps.write())
        .pipe(gulp.dest('dist'));
    });

    gulp.task('test', function (done) {
      new Server({
        configFile: 'test/karma.conf.js',
        singleRun: true
      }, done).start();
    });
    
    gulp.task('watch', ['js'], function () {
      gulp.watch('js/**/*.js', ['js', 'test']);
    });
    ```

  * the `js` task does the following:
    1. Suck every JavaScript file in `js` and its subdirectories into a stream
    2. Starts a [sourcemap](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/): a way to retain the ability to debug minified code
    3. Pass the stream through `concat`, which (surprise!) concatenates all the files into one, naming it `app.js`
    4. Pass the stream through `stripDebug`, which removes all those pesky `console.log`s
    5. Pass the stream through `uglify`, which minifies it
    6. Finishes the sourcemap
    7. Writes the stream to the destination directory.
  * The `test` task uses the [Karma](#karma) test runner to run the unit test suite
  * The `watch` task above will keep an eye on files with the `.js` suffix and run first the `js` task, then the tests, for each code change


### Further Reading

* [Gulp vs Grunt. Why one? Why the Other?](https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4)
* [Getting Started with Gulp](https://markgoodyear.com/2014/01/getting-started-with-gulp/)


## Browserify

* Bundles the dependencies for your project, letting you program with modules while browser support for _ES6_ and `import`/`export` is still patchy
* Can also handle minification and other pre-deployment tasks
* Can be used from the command line or within the context of other tools such as [Gulp](#gulp):

    ```js
    var gulp = require('gulp');
    var browserify = require('browserify');
    var babelify = require('babelify');
    var source = require('vinyl-source-stream');

    var bundler = browserify({
        entries: './src/main.js', 
        extensions: ['.js'], 
        debug: true
    });

    gulp.task('default', function () {
        return bundler
            .transform(babelify)
            .bundle()
            .pipe(source('app.js'))
            .pipe(gulp.dest('js'));
    });
    ```

  * To walk through the above `gulpfile.js`:
    1. Create a `bundler` using Browserify which begins at `main.js` and scoops up everything that app.js requires to work (dependencies)
    2. Pass it through a transform using `babelify`, which is specifically designed to create _ES5_ output from _ES6_ JavaScript
    3. Bundle it all up
    4. Pipe the result into a file named `app.js`
    5. Write `app.js` into the `js` destination directory


## Testing

### Jasmine

* A long-standing [behaviour-driven](https://en.wikipedia.org/wiki/Behavior-driven_development) test framework
* Probably as close to a 'standard' as JavaScript has, although alternatives are growing in popularity
* Tests are just JavaScript files, which can contain any code you like

    ```js
    describe('Wombat Counter', function() {
      describe('GET /', function() {

        it('responds with HTTP status 200', function() {
          request.get(base_url, function(error, response, body) {
            expect(response.statusCode.toBe(200));
          });
        });

      });
    });
    ```

* Jasmine comes with its own assertion tools:

    ```js
    expect(countTheAardvarks()).toEqual(9);
    expect("I'm a wombat").toContain("wombat");
    expect(countTheAardvarks()).not.toEqual(8);
    ```

* It also has _spies_: a way to keep tabs on parts of your program where it's not simple to track what's happening, like methods/functions

    ```js
    var a = new ArmouryOfAardvarks();
    spyOn(a, 'countTheAardvarks');
    a.showPopulation();

    // Did showPopulation call countTheAardvarks?
    expect(a.countTheAardvarks).toHaveBeenCalled();
    ```


### Mocha

* On the surface, Mocha looks very similar to Jasmine but there are some important differences:
  * Mocha doesn't have an assertion tool or spies, preferring third party libraries for this purpose
    * (see [Chai](#chai) and [Sinon](#sinon), below)
    * Mocha can be considered more flexible, but requires a bit more setup/configuration before use than Jasmine
* For many uses there will be little difference between Jasmine, Mocha, and _[ insert library here ]_.
  * The take-home message is, it's probably more important that your app has good test coverage than spending time worrying about which framework to use!


### Chai

* [Chai](http://chaijs.com) is perhaps the most popular assertion library for Mocha.
* Chai supports a variety of assertion styles and is extensible via plugins.
  * _assert_:

    ```js
    assert.typeOf(foo, 'string', 'Yup, that is a string.');
    assert.lengthOf(aardvarkArmoury, 9, 'There appear to be 9 aardvarks.');
    ```

  * _BDD_ (extremely similar to Jasmine assert style):

    ```js
    expect(foo).to.equal('bar');
    expect(aardvarkArmoury).to.have.property('population').with.length(9);
    ```

  * _should_ (extends each object with a new property):

    ```js
    aardvarkArmoury.should.have.property('population').with.length(9);
    ```


### Sinon.JS

* [Sinon](http://sinonjs.org) is a standalone spy/mocking tool, often used as [a Chai plugin](http://chaijs.com/plugins/sinon-chai).
* Example pulled from Chai site:

    ```js
    "use strict";
    var chai = require("chai");
    var sinon = require("sinon");
    var sinonChai = require("sinon-chai");
    chai.should();
    chai.use(sinonChai);

    function hello(name, cb) {
      cb("hello " + name);
    }

    describe("hello", function () {
      it("should call callback with correct greeting", function () {
        var cb = sinon.spy();

        hello("foo", cb);

        cb.should.have.been.calledWith("hello foo");
      });
    });
    ```


### Karma

* [Karma](https://karma-runner.github.io) is a test runner, commonly associated with Angular development but usable as a standalone.
  * Used to go by the unlikely moniker, "Testacular"
* Can watch for changes in code files and re-run unit tests to check if you've broken anything
* Controlled by `karma.conf.js`:

   ```js
    module.exports = function(config){
      config.set({

        basePath : './',

        files : [
          '/media/work/src/js/angular.js',
          '/media/work/src/js/angular-route.js',
          '/media/work/src/js/angular-resource.js',
          '/media/work/src/js/angular-mocks.js',
          'js/app.js',
          'js/test/*_tests.js',
        ],

        autoWatch : true,

        frameworks: ['jasmine'],

        browsers : ['Chrome'],

        plugins : [
          'karma-chrome-launcher',
          'karma-firefox-launcher',
          'karma-jasmine',
          'karma-junit-reporter'
        ],

        junitReporter : {
          outputFile: 'test_out/unit.xml',
          suite: 'unit'
        }
      });
    };
    ```


### Protractor

* [Protractor](https://angular.github.io/protractor) is an end-to-end testing framework.
  * Gives you the ability to simulate user interaction throughout your app, running in a real browser
  * Can click each link, check form validation, take screenshots, etc


### Further Reading

* [Jasmine vs. Mocha, Chai, and Sinon](http://thejsguy.com/2015/01/12/jasmine-vs-mocha-chai-and-sinon.html)

[ Index ](../README.md)
