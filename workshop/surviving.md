[ Index ](../README.md)

# Surviving in the Wild

## When is JavaScript... not JavaScript?

* JavaScript is the common/generic name of the language. 
  * EcmaScript refers to the language’s specification. 
    * ES6 refers to the 6th edition of that spec. 
    * EcmaScript 2015 is the title of the 6th edition of the spec. 
* Yes, we know this is ridiculous, but you will most certainly run across all of these on the web. 
* For the purpose of this workshop, we will use _JavaScript_ as much as possible, and _ES6_ when we are referring to a language feature that was added in the latest version of the spec only because it has fewer syllables when pronounced."
* All examples in this workshop assume _strict mode_. Incorporating [Babel](https://babeljs.io) into your workflow will take care of this for you. All _ES6_ modules are in strict mode by default.

## Function level scope

  * unlike languages with _block level scope_, in JavaScript blocks do not create their own restricted scope

    ```js
    function foo () {
      for (var i = 0; i < 10; i++) { }
    
      console.log(i); // 10
    }
    ```

  * Here, `i` is available anywhere within `foo`
  * https://jsbin.com/memopi/edit?js,console

  * In the past, was often recommended that `var` statements occur at the beginning of functions to avoid unwelcome surprises caused by this behaviour
    * With _ES6_, `var` can largely be replaced by `let`
  * The [`let`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/let) keyword restricts scope to its block, statement or expression:

    ```js
    function bar () {
      for (let i = 0; i < 10; i++) { }
    
      console.log(i); // undefined
    }
    ```
  * https://jsbin.com/warevab/edit?js,console

  * [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) creates a reference to a value that cannot be altered:
    
    ```js
    function wombat () {
      const n = 5;
      n++;            // TypeError: assignment to constant variable
    }
    ```

  * https://jsbin.com/jizivi/1/edit?js,console

## Closures

* Functions can be declared inside other functions
* local variables in the outer function are still available _even if the outer function has already returned_

    ```js
    function foo() {
      let n = 1;
      return bar;

      function bar() {
        // bar "closes over" n
        console.log(n++);
      }
    }

    let baz = foo();
    baz();              // 1
    baz();              // 2
    baz();              // 3

    let wombat = foo();
    wombat();           // 1
    baz();              // 4
    ```

  * Here, `n` remains "alive" and can be modified after `foo` exits. 
  * Each call to `foo` creates a new _closure_, with its own local copy of `n`.
  * https://jsbin.com/cibibe/1/edit?js,console


## `this`

* In _ES6_ the value of `this` is fairly predictable
  * Watch out for legacy JS, where it can unexpectedly default to the window object and lead to some unintuitive behaviour
  * Best practice is to _be explicit_: never assume that the next developer to read your code understands the nuances of `this`
  * Think _functionally_: avoid altering global or object level state

* Value of `this` depends on the execution context:

    ```js
    console.log(this);   // Value of 'this' already set to the Window object
    
    function foo () {
      console.log(this); // undefined
    }

    let bar = {
      wombat: function () {
        console.log(this); // Value of this is the object the method is called on
      }
    };
    ```

  * In the last example, `this` is the object `bar`
* `this` can be tricky when used in event handlers. Generally its value is the DOM element which fired the event:

    ```js
    <p id="foo">Aardvark.</p>

    <script>
      let foo = document.getElementById('foo');
      foo.onclick = makeGreen;

      function makeGreen() {
        // 'this' is the <p> DOM element
        this.style.backgroundColor = 'green';
      }
    </script>
    ```

    * However, each function has its own `this`, which can lead to tricky situations with surrounding functions:

    ```js
    <p id="foo">Aardvark.</p>

    <script>
      let foo = document.getElementById('foo');
      foo.onclick = toggleGreen;

      function toggleGreen() {
        if (this.style.backgroundColor === 'green') {
          makeTransparent();
        } else {
          makeGreen();
        }

        function makeGreen() {
          // 'this' is undefined
          this.style.backgroundColor = 'green'; 
        }

        function makeTransparent() {
          // 'this' is undefined
          this.style.backgroundColor = 'transparent';
        }
      }
    </script>
    ```

* Use of [`call`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) invokes a function with the desired value of `this`:

    ```js
    if (this.style.backgroundColor === 'green') {
      makeTransparent.call(this);
    }
    ```

  * Compare [`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) and [`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

  * Cross-browser issues (particularly with earlier versions of Internet Explorer) can yield surprising results, with `this` being set to the global window object.

## The object prototype

* In JavaScript every object has its own _prototype_
  * Each prototype has its own prototype, and so on down the chain until `null` is reached
  * Properties are first looked for in the object itself (see: [`hasOwnProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty), then in its prototype, then in the prototype's prototype, and so on

* It's useful to remember the difference between giving one distinct object a property:

    ```js
    let foo = {
      aardvarks: function() { 
        console.log("I see many aardvarks.");
      }
    };

    foo.aardvarks(); // I see many aardvarks
    ```

* and giving all objects created with a particular constructor function access to the property:

    ```js
    function AardvarkCounter(n) {
      this.count = n;
    }

    AardvarkCounter.prototype.aardvarks = function() {
      console.log(`I see ${this.count} aardvarks.`);
    };

    let firstCounter = new AardvarkCounter(11);
    let secondCounter = new AardvarkCounter(1337);

    firstCounter.aardvarks();  // "I see 11 aardvarks."
    secondCounter.aardvarks(); // "I see 1337 aardvarks."
    ```
* https://jsbin.com/kohese/1/edit?js,console

* Functions declared on the prototype are shared between all objects with that prototype
* Many libraries include the ability to copy properties from one object to another
* For example, using [Underscore's](http://underscorejs.org) [`extend`](http://underscorejs.org/#extend):

    ```js
    let foo = {
      n: 5
    };

    let bar = {
      wombat: function() {
        console.log("Wombats: accept no substitute.");
      }
    };

    let combined = _.extend(foo, bar);
    combined.wombat();
    console.log(combined.n); // 5
    ```


## Modules and Imports

* To make a property, function, or object available outside its own file (_module_), prefix it with the [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) keyword:
_
    ```js
    // wombats.js

    export const metabolism = 4;

    export function showWombats() {
      // ...
    };
    ```

* To make use of this in another module, we use [ `import` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import). The following will all make `showWombats` available:

    ```js
    import wombats from './wombats';             // Import the entire module
    import {showWombats} from './wombats';       // Just one function
    import {showWombats as sw} from './wombats'; // Same as above, but give it a shorter alias
    ```

  * In the first statement, `showWombats` would be referred to as `wombats.showWombats`
  * https://gist.github.com/locksmithdon/96fb0ae2904f86c86e66


## Underscore and Lo-Dash

* If you've ever wished that JavaScript had <em>X</em> feature from another language, chances are it (or something like it) is provided by a utility library
* One of the key roles of a utility library is making it simpler, more expressive, and less error-prone to work with data structures (an array, object, or string).
* Utility libraries act as extensions to the JavaScript standard library, providing consistent and well-tested ways to interact with data structures.
* The two most popular are [ Underscore ](http://underscorejs.org) and its API-compatible alternative, [ Lo-Dash ](http://lodash.com)
  - The question of which one to use has gotten quite political, and beyond our scope here. Either one will run the examples.

* Given an array:

    ```js
    let aardvarks = ["Snuffler", "Stinker", "Slappy", "Sadface"];
    ```

* You could do this:

    ```js
    for (let i = 0; i < aardvarks.length; i++) {
      console.log(aardvarks[i]);
    }
    ```

* Or this:

    ```js
    _.each(aardvarks, console.log);
    ```

* Checking to see if an object has a particular property is another common use case:

    ```js
    let wombat = {
      registeredToVote: false,
      highlyOpinionated: true
    };

    if (_.contains(_.keys(wombat), 'registeredToVote')) {
      console.log("Marsupial voter registration data exists for this entity.");
    }
    ```

* Some standard library functions are duplicated in the utility libraries: 

    ```js
    let foo = [1, 2, 3, 4, 5];

    // Standard library
    let sum1 = foo.reduce( function (prev, curr) {
      return prev + curr;
    });

    // Underscore / Lo-Dash
    let sum2 = _.reduce(foo, function (total, n) {
      return total + n;
    });

    // Alternative syntax
    let sum3 = _(foo).reduce( function (total, n) {
      return total + n;
    });

    console.log(sum1, sum2, sum3); // 15 15 15
    ```

* Why not use the standard library version? 
  * Allows us to use the same approach whenever working with any data structures (objects, strings, and arrays), so we don't need to explicitly convert to array and back again. This can give our code consistency and readibility.
  * In some circumstances, the library will default to the native version if it's known to be faster in a given environment.


## Further Reading

* [let - MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/let)
* [const - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
* [Closures - canonical Stack Overflow answer](http://stackoverflow.com/a/111111/122643)
* [this - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
* [all this](http://bjorn.tipling.com/all-this)
* [JavaScript 'this' and event handlers](http://www.sitepoint.com/javascript-this-event-handlers/)
* [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
* [Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
* [Promise - MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [The Long Road to Async/Await in JavaScript](https://thomashunter.name/blog/the-long-road-to-asyncawait-in-javascript/)
* [Lodash: 10 JavaScript Utility Functions That You Should Probably Stop Rewriting](http://colintoh.com/blog/lodash-10-javascript-utility-functions-stop-rewriting)
* [Learn ES2015](https://babeljs.io/docs/learn-es2015/)
* [Exploring ES6 (free online book)](http://exploringjs.com/es6/)

[ Index ](../README.md)
