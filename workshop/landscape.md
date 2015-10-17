[ Index ](../README.md)

# The Landscape

## Why JavaScript?

* JS API consumption predates JS API provision by about 10 years. Asynchronous content loads date from late 90's, term AJAX coined in 2005
* Server-side JavaScript using Node since 2009

### Immediacy:
* Supremely easy to get started, essentially zero cost entry point...
* Potentially same developers on front and back end
* Quick design-code-test-debug cycles

### Reliability:
* Infrastructure investments (Google -> V8, Microsoft -> Chakra)
* Large, mainstream frameworks (Google -> Angular, Facebook -> React)
* Thriving community (Node.js -> IO.js -> 4.0, and NPM modules)
* As of Oct 2015, over 195K NPM packages

### Portability:
* Not only capable of building frontend and backend web, but mobile and desktop
* Blurring lines between browser and native (React Native, Phonegap)
* No longer the wild west: *ES6*

### Maintainability:
* Simple, well-established mechanisms for sharing and troubleshooting code (REPLs everywhere: jsfiddle.net, jsbin.com, plnkr.io, codepen.io)
* _Lingua franca_ effect: finding JS devs not difficult (quality variable)
* Reasonable debugging tools using the browser and nodemon
* Voracious (and opinionated) community
 

## Why not JavaScript?

### Tooling 
* Even today, with WebStorm, and Visual Studio incorporating JS debugging, still not as good as statically typed languages
* Can be hard to debug, particularly larger code bases

### Framework churn
* Makes assessing the correct tool for a particular task more challenging
* May mean poorer documentation, out of date tutorials, lost time

### Speed
* Traditional response to speed concerns around JS is, "processing speed always increases"
* Compare with Go and other compiled backend languages

### Regional preference
* Full-stack JavaScript perhaps less common in New Zealand
  * May mean delays in finding developers for support/maintenance, although this is changing


## Why Web APIs?

To be clear, by web APIs we're referring to HTTP calls to a REST service that operates on resources.

### Decoupled
* UX can be developed independently
  * Arbitrary partitioning: microservices or monolithic
  * Arbitrary language/stack, everything groks JSON
* Common formats (JSON: .doc for the web?)
* Can be versioned independently

### Expose public data sets
* Independent of consuming applications
* Consumers at organisational level, paying clients, or open
* Supports OAuth authentication

### Clear expectations
* Simple and straight-forward requests response model
* Discovery through HATEOAS Level 3 of the Richardson Maturity Model
  - HATEOAS: Hypertext As The Engine Of Application State
* Good debugging tools like Postman


## Why Not Web APIs?
* Every request across HTTP is a time sink

## Further reading

* [JavaScript will lead a massive shift in enterprise development](http://www.infoworld.com/article/2907190/javascript/javascript-will-lead-a-massive-shift-in-enterprise-development.html)

[ Index ](../README.md)
