# Express.js routing

## What is routing?

Routing is a way to map requests to specific handlers depending on their URI and HTTP verb.

When using a browser to visit example.com/jumpstart, the raw HTTP request might look like this:

```
GET /jumpstart http/1.1
```

The HTTP request consist of a verb (GET), a URI (/jumpstart), and the HTTP version (1.1). When we are routing, we take the verb and the URI and map it to a handler function. In this case, when Express sees a GET request to /jumpstart, some code is run.

A browser (the client) can make a GET, POST, PUT or DELETE HTTP request for various URLs (e.g. GET http://localhost:3000/books or DELETE http://localhost:3000/students/42).

Make sure to understand more about [HTTP](../fundamentals/http) before learning about routing.

Refer to the script: [Express.js playground: express_basic_example_1](https://github.com/thoughtworks-jumpstart/express-playground/blob/master/express_basic_example_1.js)

## Request methods

A request method is derived from one of the HTTP methods (also known as HTTP verbs) like GET, POST and PUT etc.

Route that is defined for the GET method to the root of the app:

```js
// express_basic_example_1.js
app.get("/", (req, res) => {
  res.send("Welcome to my homepage");
});
```

Route that is defined for the POST method to the root of the app:

```js
app.post("/", (req, res) => {
  res.send("POST request to the homepage");
});
```

The handler function (callback function) is called when a GET request is sent by the browser to the root `/`. In the handler function, "Welcome to my homepage" is sent as the response.

Express.js will call the handler function with two parameters. The first parameter is the request object and the second parameter is the response object.

## Route paths

Route paths, in combination with a request method, define the endpoints at which requests can be made. Route paths can be strings, string patterns, or regular expressions.

### Fixed route path

The handler function is called when a GET request is sent by the browser to the fixed route `/books`. In the handler function, "You requested a list of books...." is sent as the response.

```js
app.get("/books", (req, res) => {
  res.send("You requested a list of books....");
});
```

### Regular expressions

This route path will match /books and /book.

```js
app.get("/books?", function(req, res) {
  res.send("You requested a list of books...");
});
```

This route path will match butterfly and dragonfly, but not butterflyman, dragonflyman, and so on.

```js
app.get(/.*fly$/, function(req, res) {
  res.send("/.*fly$/");
});
```

### Route parameters

We can grab data from the route using route parameters. Route parameters are named segments of the URL used to capture values at their specified position.

GET a specific book with ID:

```
Route path: /books/:bookId
Request URL: http://localhost:3000/books/8989
req.params: {"bookId": "8989" }
```

```js
app.get("/books/:bookId", (req, res) => {
  res.send(`You requested information on book ${req.params.bookId}`);
  // NOTE: the above line poses a security issue, we should always treat any user input as unsafe (see XSS attack)
});
```

This route will match paths `/books/1`, `/books/2` and so on. Will it match `/books/abc`?

`bookId` is used to capture values at that position of the path. Thus `req.params` has a property called `bookId` storing `1` or `2` etc depending on the path.

The captured values are populated in the `req.params` object, with the name of the route parameter specified in the path as their respective keys.

it is possible to capture more than one value using multiple parameters.

```
Route path: /users/:userId/books/:bookId
Request URL: http://localhost:3000/users/34/books/8989
req.params: { "userId": "34", "bookId": "8989" }
```

```js
app.get("/users/:userId/books/:bookId", (req, res) => {
  //...
});
```

### Route querystrings

Refer to the script: [Express.js playground: query_parameter_example](https://github.com/thoughtworks-jumpstart/express-playground/blob/master/query_parameter_example.js).

We can use querystrings to filter data, provide information to a page or even alter the action of a page.

Considering that we have the following data:

```js
const data = [
  {
    id: 1,
    type: "FRUIT",
    name: "Banana",
    color: "yellow",
  },
  {
    id: 2,
    type: "FRUIT",
    name: "Tomato",
    color: "red",
  },
  {
    id: 3,
    type: "VEGETABLE",
    name: "Broccoli",
    color: "green",
  },
  {
    id: 4,
    type: "VEGETABLE",
    name: "Red pepper",
    color: "red",
  },
];
```

These four items have a few properties / fields that we can use to filter the actual items that we return to the user.

If we only want to get items which type is FRUIT:

```
Route path: /food
Request URL: http://localhost:3000/food?type=FRUIT
req.query: { "type": "FRUIT" }
```

If we only want to get items which color is red:

```
Route path: /food
Request URL: http://localhost:3000/food?color=red
req.query: { "color": "red" }
```

If we want to get items which color is red and type is also FRUIT:

```
Route path: /food
Request URL: http://localhost:3000/food?color=red&type=FRUIT
req.query: { "color": "red", "type": "FRUIT" }
```

```js
app.get("/food", (req, res) => {
  //console.log(req.query);

  const results = data
    // Using if statements
    .filter(item => {
      if (req.query.type) {
        return item.type === req.query.type;
      }

      return true;
    })
    // Using the ternary operator
    .filter(item => (req.query.name ? item.name === req.query.name : true))
    .filter(item => (req.query.color ? item.color === req.query.color : true));

  res.json(results);
});
```

The querystrings are populated in the `req.query` object, with the name of the query as their respective keys. What do you think the above code does?

## Multiple handler functions

Refer to the script: [Express.js playground: express_basic_example_2](https://github.com/thoughtworks-jumpstart/express-playground/blob/master/express_basic_example_2.js).

Handler function 1:

```js
// express_basic_example_2.js
const requestHandler1 = (req, res, next) => {
  res.write("Here is a list of students:\n");
  next();
};
```

Handler function 2:

```js
const requestHandler2 = (req, res) => {
  res.write("Jesstern\n");
  res.write("Elson\n");
  res.write("Mabel\n");
  res.write("Bye!\n");
  res.end();
};
```

The first handler function passes the request to the next handler function via `next()` call.

When we are not ready to send back the response yet, we use the `res.write()` method to update the response, instead of using the `res.send()` method.

If a handler function needs to send the response back to client, it should call the `res.end()` method.

More than one handler function can be defined for a route. You can define the above two handler functions for the same route.

You can call `app.METHOD()` multiple times:

```js
app.get(path, requestHandler1);
app.get(path, requestHandler2);
```

or you can declare the handlers in one line:

```js
app.get("/students", requestHandler1, requestHandler2);
```

Note that the handlers are executed in the same order as declaration. In the example above, requestHandler1 is called before requestHandler2.

If you send a request to http://localhost:3000/students, you should see the output of both handler functions, printed in the right sequence.

For more examples, see the [express docs](https://expressjs.com/tr/guide/routing.html).
