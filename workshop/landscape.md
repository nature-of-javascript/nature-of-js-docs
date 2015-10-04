[ Index ](../README.md)

# The Landscape

## Why JavaScript?

* JS API consumption predates JS API provision by about 10 years. Asynchronous content loads date from late 90's, term AJAX coined in 2005
* Node from 2009: server-side JS

### Immediacy:
* Supremely easy to get started, essentially zero cost entry point...
  * ...but like all projects, potentially expensive to maintain
* Quick design-code-test-debug cycles
* Potentially same developers on front and back end

### Reliability:
* Large, mainstream frameworks (Google -> Angular, Facebook -> React)
* Infrastructure investments (Google -> V8, Microsoft -> Chakra)
* Thriving community (Node.js -> IO.js -> 4.0, and NPM modules)

### Portability:
* Blurring lines between browser and native (React Native, Phonegap)
* No longer the wild west: *ES6*

### Maintainability:
* Simple, well-established mechanisms for sharing and troubleshooting code (REPLs everywhere)
* Voracious (and opinionated) community
* _Lingua franca_ effect: finding JS devs not difficult (quality variable)
 

## Why Not JavaScript?

### Tooling 
* even today, with WebStorm, and Visual Studio incorporating JS debugging, still nowhere near as good as those for statically typed languages
* Can be hard to debug, particularly larger code bases

### Framework Churn
* makes assessing the correct tool for your current job more challenging
* may mean poorer documentation, out of date tutorials, lost time

### Speed
* Traditional response to speed concerns around JS is, "processing speed always increases"
* Compare with eg Go, other compiled backend languages

### Regional Preference
* Full-stack JavaScript perhaps less common in New Zealand
  * May mean delays in finding developers for support/maintenance, although this is changing


## Why APIs?

### Decoupled
* UX can be developed independently
  * Arbitrary partitioning: microservices or monolithic
  * Arbitrary language/stack, everything groks JSON
* Versioning
* Common formats (JSON: .doc for the web?)

### Expose public data sets
* Independent of current application
* Consumers at organisational level, paying clients, or open

### Guarantees
* Contract with consumers: if you format your request _this way_, your data will always look _like this_


## Why Not APIs?
* Every request across HTTP is a time sink

## Further Reading

* [JavaScript will lead a massive shift in enterprise development](http://www.infoworld.com/article/2907190/javascript/javascript-will-lead-a-massive-shift-in-enterprise-development.html)

[ Index ](../README.md)
