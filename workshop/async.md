# Async JavaScript

## Callbacks
* Execute code asynchronously: simply put, _later_.
* Typically passed to another function, to be used when an event occurs, or when the function has completed

    ```js
    function displayResults(results) {
      for (let i = 0; i < results.length; i++) {
        console.log(results[i]);
      }
    }

    function incrementAll(numbers, callback) {
      for (let i = 0; i < numbers.length; i++) {
        numbers[i] += 1;
      }
      callback(numbers);
    }

    incrementAll([1, 2, 3, 4, 5], displayResults);
    ```

  * Here, the function `displayResults` is passed as a _callback_ to `incrementAll`.
  * It outputs values to the console, but we could just as easily pass in a function which further modifies the array

* Many JavaScript standard library functions use callbacks, often optionally. For example:

    ```js
    function even(n) {
      return n % 2;
    }

    console.log([1, 2, 3, 4, 5].filter(even)); // [1, 3, 5]
    ```

  * `even` serves as a callback to [`Array.filter`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

* Callbacks can be inline, which should only be used if it improves readability (i.e. not for lengthy callbacks)

    ```js
    console.log([1, 2, 3, 4, 5].filter( function () { return n % 2; } );
    ```


## Promises

* A _promise_ is an operation that we expect to complete at a future time
* Allows us to guarantee order of execution in an asynchronous language

    ```js
    function showWombats() {
      let wombats = getWombatsFromServer();
      displayWombats(wombats);
    }
    ```

  * In a synchronous language, program execution will pause until `getWombatsFromServer` returns. In JavaScript, execution will continue
    * Unless `getWombatsFromServer` returns almost instantaneously, `wombats` will be undefined and attempting to display it will have unpredictable results

* Consider the same function using promises:

    ```js
    function showWombats() {
      getWombatsFromServer().then(displayWombats);
    }

    function displayWombats(data) {
      // ...
    }
    ```

  * Here we guarantee that the call to display won't occur until after the server request has returned, even if it takes several seconds to complete
* Calls to `then` can be chained to further control the order of execution:

    ```js
    getWombatsFromServer()
      .then(displayWombats)
      .then(function () {
        console.log("All wombats present and accounted for.");
      });
    ```

* There's plenty more to promises, including the ability to handle failure gracefully using [`catch`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch). 


## `async`, `await`

* In _ES7_ (yep, another version!) the `async`/`await` keywords provide another approach to asynchronous code

    ```js
    async function showWombats() {
      let wombats = await getWombatsFromServer();
      displayWombats(wombats);
    }
    ```

  * Users of async/await in C# will find the syntax very familiar
  * Function execution only continues when the `await` resolves


