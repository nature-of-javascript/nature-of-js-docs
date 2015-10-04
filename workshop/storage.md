[ Index ](../syllabus.md)

# Storage

## MongoDB

* Although in theory JavaScript can interact with almost any data store, [Mongo](https://www.mongodb.org) is one of the most common databases in modern JS stacks.


### NoSQL

* Most traditional databases are based on tables, and the relationships between them.
* NoSQL databases come in a variety of designs. Mongo is of the _document store_ type.
  * Stores records in JSON format, encoded to binary format _BSON_
  * Tends to lump all related information together, rather than distribute it across relational tables
  * Although each document can be validated in code, they aren't required to be identical in structure
* Mongo arranges documents into 'collections'. A database is made up of a set of collections.


### JSON

* Do yourself a favour and read certainly [the best JSON documentation on the Internet](http://json.org).
* Mongo happily accepts and dispenses documents as JavaScript Object Notation.
* If a record is returned in the body of a response, it's not even necessary to call `JSON.parse` on it... it's already usable as a JavaScript object.
* Mongo will add an `ObjectId` (`_id`) property automatically, and possibly versioning information depending on settings:

    ```js
    {
      "__v": 0,
      "_id": "55fb213fac8d99ce3fb1b47c",
      "name": "Fitzgerald",
      "stomachContents": [
        "ant", "ant", "termite", "ant"
      ],
      "latinName": {
        "genus": "Orycteropus",
        "species": "afer"
      },
      "nocturnal": true
    }
    ```

## Mongoose

* [Mongoose](http://mongoosejs.com) is an object model for [MongoDB](#mongodb).


### Schemas

* Mongoose gives each document a _schema_, one schema per Mongo _collection_.
  * Allows types, defaults, and opportunity to define static methods and middleware.
* Schemas can contain other schemas:

    ```js
    let itemSchema = new Schema({
      name: String,
      color: String
    });

    let containerSchema = new Schema({
      name: String,
      items: [itemSchema]
    });
    ```

### Models

* As schemas are to MongoDB collections, _models_ are to MongoDB _documents_.
* Models are created using a schema:

    ```js
    let Wombat = mongoose.model('Wombat', wombatSchema);
    ```

* Adding a document to the collection in the database requires creating a new instance of the model:

    ```js
    let myWombat = new Wombat({
      name: "Holiday"
    });
    myWombat.save(function (err) {
      if (err) return handleError(err);
    });
    ```

* Schemas provide various methods for finding, altering and deleting documents:

    ```js
    Wombat.findOne({ 'name': 'Holiday' }, 'name', function (err, wombat) {
      if (err) return handleError(err);
      console.log('Found her!');
    });
    ```

  * See [Queries](http://mongoosejs.com/docs/queries.html) for further details.


### Validation

* One of the strongest reasons for using Mongoose is that validation is built in.
  * For example, although properties are considered optional by default, they can be made required:

    ```js
    import mongoose, {Schema} from 'mongoose';

    let wombatSchema = new Schema({
      name: { type: String, required: true },
      nocturnal: Boolean
    });
    ```

  * Above, `name` must be supplied but `nocturnal` can be omitted. Attempting to save a `Wombat` without a `name` string will cause a validation error.


## Database-backed endpoints

* Having established a means to store and retrieve data, the REST API serves as an HTTP interface between the client and the data store.


### Characteristics

* The REST API operates on the assumption that the client will be accessing the same base URL or _resource_ (sometimes with additional parameters like ID numbers) but using different _HTTP verbs_.
  * The server can return data in the _response body_, but it can also provide feedback in the form of the HTTP _status code_, and in 'strict' REST it should provide all the information necessary to navigate the site's resources.
    * [What is HATEOAS and why is it important for my REST API?](http://restcookbook.com/Basics/hateoas/) and [Haters gonna HATEOAS](http://www.timelessrepo.com/haters-gonna-hateoas) for further context.
* For example:

    ```
    GET /wombats
      - The same as visiting the resource in a browser
      - HTTP status 200 for a successful request

    GET /wombats/1
      - Return a single wombat with id 1
      - 200 for success
      - 404 not found

    POST /wombats
      - Create a wombat
      - 201 for resource created, and the response body should contain the wombat

    PUT /wombats/1
      - Edit or update the wombat with id 1
      - 200 for resource edited, or 204 if the body doesn't contain the wombat
    
    DELETE /wombats/1
      - Remove the wombat with id 1
      - 200 for resource deleted, or 204 if the body doesn't contain the deleted resource
    ```

* In each case the address is the same or very similar, but the _HTTP verb_ is different.
* This allows the server to behave differently depending on the request. For example, using Express:

    ```js
    router.route('/wombats')

      // GET /wombats
      .get(function (req, res) {
        Wombat.find(function (err, wombats) {
          respond(res, err, wombats);
        });
      })

      // POST /wombats
      .post(function (req, res) {
        let wombat = new Wombat();
        wombat.name = req.body.name;
        wombat.save(function (err) {
          respond(res, err, wombat);
        });
      });

    function respond(res, err, wombats) {
      if (err) {
        res.send(err);
      } else {
        res.json(wombats);
      }
    }
    ```

  * Here, Express uses a different function for GET and POST verbs even though the address visited is identical.
    * In the case of GET, since there is no ID specified, it simply returns an array of wombats (as JSON).
    * For POST, it looks for a wombat in `req.body`, checks the `name` specified and saves it to MongoDB.

* We can also tell Express to respond differently if a resource ID has been specified:

    ```js
    router.route('/wombats/:id')

      // GET /wombats/12345
      .get(function (req, res) {
        Wombat.findById(req.params.id, function (err, wombat) { 
          respond(res, err, wombat); 
        });
      })

      // PUT /wombats/12345
      .put(function (req, res) {
        Wombat.findByIdAndUpdate(req.params.id, req.body, function (err, wombat) {
          respond(res, err, wombat);
        });
      })

      // DELETE /wombats/12345
      .delete(function (req, res) {
        Wombat.findByIdAndRemove(req.params.id, req.body, function (err, wombat) {
          respond(res, err, wombat);
        });
      });
    ```

  * Express places the `id` in `req.params.id`.
  * The functions `findById`, `findByIdAndUpdate` and `findByIdAndRemove` are provided by Mongoose.


### Async Implications

* Requests to the database take place in unknown time
  * The database or server could be under heavy load
  * The network infrastructure may be impaired
  * Further cloud resources (additional servers, containers, database instances) may be in the process of coming online
* We cannot _know_ when the request will return with data (or an error)
* We need a way to guarantee that the values we're expecting won't be accessed until after the call to the database has completed

    ```js
    router.route('/wombats/:id').get(retrieveOneWombat);

    function retrieveOneWombat(req, res) {
      Wombat.findById(req.params.id, sendWombatResponse);
    }

    function sendWombatResponse(err, wombat) {
      if (err) {
        res.send(err);
      } else {
        res.json(wombat);
      }
    }
    ```

  * This is a simple re-working of the sample code in the previous section, avoiding the use of inline functions for clarity.
  * There are two callbacks: `retrieveOneWombat` and `sendWombatResponse`.
    * When Express receives a request to this URL (resource) with the HTTP verb `GET`, it calls `retrieveOneWombat`
    * When the Mongoose model `Wombat`'s method `findById` has finished hitting the database, it calls `sendWombatResponse`
  * This gives us some assurance that our data is actually there, or if it isn't that an error has been generated which we can return.

* This is callback-based async code. Newer versions of Mongoose also support promises, so you can use:

    ```js
    router.route('/wombats/:id')
      .get(function (req, res) {
        Wombat
          .findById(req.params.id)
          .then(function (wombat) {
            // ...
            res.json(wombat);
          });
      });
    ```


[ Index ](../syllabus.md)
