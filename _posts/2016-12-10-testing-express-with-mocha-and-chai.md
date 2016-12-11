---
layout: post
title:  "Testing Express with Mocha and Chai"
date:   2016-12-10
categories: development
---

In this post we are going to create a server app using Express, and we'll test it with Mocha and Chai. Testing our app allows us to keep control of every change in the code while we are in the development stage.

## Building our app

We'll write a program that listens to requests, and responds appropriately. When it receives a `POST` request for `/set?somekey=somevalue` it will store the passed key and value in memory. When it receives a `GET` request for `/get?key=somekey` it will return the value stored at `somekey`.

Let's create a directory to hold our app, and make it our working directory:

```
$ mkdir app
$ cd app
```

Then we'll run `npm init -y` to create a `package.json` file. This file will list all the dependencies necessary for our project, as well as some other metadata.

We can install [Express](http://expressjs.com/) by running:

```
$ npm install --save express
```

This command will also include the module in our `package.json` file as a dependency. It will look like this:


```json
{
  "name": "app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.14.0"
  }
}
```

Let's check that everything is working by creating the simplest Express app in a file called `app.js`:

```js
var express = require('express')
var app = express()

app.get('/', function (req, res) {
  res.send('Hello World!')
})

app.listen(4000, function () {
  console.log('We are listening on port 4000!')
})
```

We can start our server app by running:

```
$ node app.js
```

It will start listening on port 4000. We can send it a request by visiting <http://localhost:4000> in our browser, or using `curl`:

```
$ curl localhost:4000
Hello world!
```

As it's pretty common to start server apps using `npm start` instead of specifying the filename directly, we're going to add that shortcut to our `package.json`:

```json
{
  "name": "app",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.14.0"
  }
}
```

Ok, now it's time to write some tests.

## Test-driving our app

We are going to test-drive our server app using [Mocha](https://mochajs.org/) and [Chai](http://chaijs.com/). Mocha is a feature-rich JavaScript test framework running on Node and in the browser, and it's specially convenient for asynchronous testing.

We'll install `mocha` as a development dependency for our project:

```
$ npm install --save-dev mocha
```

Chai is an assertion library that integrates very well with Mocha. Let's install it:

```
$ npm install --save-dev chai
```

Since we are going to make requests to our server, we'll also install [chai-http](https://github.com/chaijs/chai-http) to have nicer assertions:

```
$ npm install --save-dev chai-http
```

Let's also set things up so that running `npm test` will run our tests with `mocha`. Our `package.json` will now look like this:

```json
{
  "name": "app",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js",
    "test": "mocha"
  },
  "dependencies": {
    "express": "^4.14.0"
  },
  "devDependencies": {
    "chai": "^3.5.0",
    "chai-http": "^3.0.0",
    "mocha": "^3.2.0"
  }
}
```

Now it's time to write our first test and see Mocha in action. We'll create a new file `test/app_spec.js`:

```
mkdir test
touch test/app_spec.js
```

## Testing our first endpoint

Our first test will check that our `POST` requests to `/set` are received correctly, and the response status is `200`:

```js
// test/app_spec.js

var chai = require('chai');
var chaiHttp = require('chai-http');
var app = require('../app');

var expect = chai.expect;

chai.use(chaiHttp);

describe('App', function() {
  describe('/set?somekey=somevalue', function() {
    it('responds with status 200', function(done) {
      chai.request(app)
        .post('/set?somekey=somevalue')
        .end(function(err, res) {
          expect(res).to.have.status(200);
          done();
        });
    });
  });
});
```

If we try to run our tests with `npm test`, we'll encounter this error:

```
TypeError: app.address is not a function
 at serverAddress (node_modules/chai-http/lib/request.js:252:18)
 at new Test (node_modules/chai-http/lib/request.js:244:53)
 at Object.obj.(anonymous function) [as post] (node_modules/chai-http/lib/request.js:216:14)
 at Context.<anonymous> (test/app_spec.js:13:10)
```

This is because our `app.js` file is being required in the test, but isn't exporting anything. Let's fix that:

```js
// app.js

var express = require('express')
var app = express()

// ...

module.exports = app
```

If we run our test again, it will fail with the expected error:

```
Uncaught AssertionError: expected { Object (domain, _events, ...) } to have status code 200 but got 404
  at test/app_spec.js:15:31
  at Test.Request.callback (node_modules/superagent/lib/node/index.js:631:3)
  at IncomingMessage.<anonymous> (node_modules/superagent/lib/node/index.js:795:18)
  at endReadableNT (_stream_readable.js:974:12)
  at _combinedTickCallback (internal/process/next_tick.js:74:11)
  at process._tickCallback (internal/process/next_tick.js:98:9)
```

Now let's implement the `/set` endpoint. We'll receive the key and value in the query part of the request, and we'll store them in a hash in memory:

```js
// app.js

var express = require('express');
var app = express();

var cache = {};

app.post('/set', function(req, res) {
  var query = req.query;
  Object.keys(query).forEach(function(key) {
    cache[key] = query[key];
  });
  res.status(200).end();
});

module.exports = app;
```

If we run `npm test` now, everything will be green. 👍

## Testing our second endpoint

We'll write our second test to check that `GET` requests to our `/get` endpoint are received correctly, and respond with status `200`. We'll also check that the endpoint returns the value stored in a previous request:

```js
// test/app_spec.js

describe('App', function() {
  // ...

  describe('/get', function() {
    it('responds with status 200', function(done) {
      chai.request(app)
        .post('/set?somekey=somevalue')
        .then(function() {
          chai.request(app)
            .get('/get?key=somekey')
            .end(function(err, res) {
              expect(res).to.have.status(200);
              expect(res.text).to.equal('somevalue');
              done();
            });
        });
    });
  });
});
```

If we run `npm test` at this point, we'll get the following error:

```
Uncaught AssertionError: expected { Object (domain, _events, ...) } to have status code 200 but got 404
  at test/app_spec.js:29:35
  at Test.Request.callback (node_modules/superagent/lib/node/index.js:631:3)
  at IncomingMessage.<anonymous> (node_modules/superagent/lib/node/index.js:795:18)
  at endReadableNT (_stream_readable.js:974:12)
  at _combinedTickCallback (internal/process/next_tick.js:74:11)
  at process._tickCallback (internal/process/next_tick.js:98:9)
```

Let's implement the `/get` endpoint. It just needs to return whatever is in our `cache` for the `key` specified in the request query:

```js
// app.js

var express = require('express');
var app = express();

var cache = {};

app.post('/set', function(req, res) {
  // ...
});

app.get('/get', function(req, res) {
  res.send(cache[req.query.key]);
});

module.exports = app;
```

Everything's green again! 👍👍

Now we have a working server listening on port `4000`. We can store keys and their values in a hash, and these values can be accessed via their keys very easily. Awesome!
