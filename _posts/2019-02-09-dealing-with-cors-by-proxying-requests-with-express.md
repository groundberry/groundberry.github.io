---
layout: post
title:  "Dealing with CORS by proxying requests with Express"
date:   2019-02-09
categories: development
canonical: https://www.telerik.com/blogs/supporting-cors-by-proxying-requests-with-express
---

In our last article, we built a [React](https://facebook.github.io/react/) app to render historical stock prices. Everything was working fine, until we encountered an issue when trying to hit the [IEX API](https://iextrading.com/developer/docs/).

Our frontend (<http://localhost:3000>) and the IEX API (<https://api.iextrading.com/>) live under different domains, so the browser applies the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) and blocks any request going from the former to the latter.

In this article we'll learn how to overcome this issue by building a proxy with [Express](https://expressjs.com/) that will request stock data from the [IEX API](https://iextrading.com/developer/docs/) on our behalf, and will emit the right CORS headers so that our frontend can access it without problems.

{% include image.html name="stock-chart-app.png" alt="StockChart app" %}

You can see the code for the proxy we'll be building on [this GitHub repo](https://github.com/groundberry/stock-chart-proxy-express).

## What is CORS?

*Browser don't hurt me, don't hurt me... no more.* 😅

The [MDN documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) defines it like this:

> **CORS** (Cross-Origin Resource Sharing) is a mechanism that uses additional HTTP headers to tell a browser to let a web application running at one origin (domain) have permission to access selected resources from a server at a different origin. A web application makes a cross-origin HTTP request when it requests a resource that has a different origin (domain, protocol, and port) than its own origin.

Given that CORS headers need to be included in the response we get from the server, and given that we have no control over the IEX API, what we'll need to do is build our own proxy server that forwards requests to the IEX API, but emits the right CORS headers so that the browser doesn't block anything. Then we'll need to make changes to our client so that it hits our server instead of IEX's.

## Building the proxy server

We'll build our proxy server using [`express`](https://expressjs.com/), a really popular Node web application framework. We'll make requests to the IEX API using [`request`](https://github.com/request/request), a simple HTTP client. During development, we'll also use [`nodemon`](https://nodemon.io/) to monitor for any changes in our source and automatically restart the server, which will save us a ton of time.

Let's start our project and install all those dependencies:

```
$ mkdir stock-chart-proxy-express
$ cd stock-chart-proxy-express
$ npm init -y
$ npm install --save express request
$ npm install --save-dev nodemon
```

While we're at it, we can configure the `start` script in our `package.json` to run `nodemon`:

```json
// package.json

{
  "name": "stock-chart-proxy-express",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.16.4",
    "request": "^2.88.0"
  },
  "devDependencies": {
    "nodemon": "^1.18.9"
  }
}
```

We'll then write the simplest server possible in that `server.js` file:

```js
// server.js 

const express = require('express');
const app = express();
const port = 3001;

app.get('/', (req, res) => res.send('Hello World!'));

app.listen(port, () => console.log(`http://localhost:${port}`));
```

If we `npm start` and access <http://localhost:3001> with our browser, we'll see `Hello World!` on the screen. If you make any changes to the `server.js` file now, you'll see `nodemon` restarting the server automatically. Success!

### Making requests to the IEX API

Our proxy server will listen to our requests, and forward them to the IEX API. Even though the IEX API supports a ton of options, we only care about two pieces of information: the symbol the user entered, and the date range the user selected.

We'll read those two parameters `symbol` and `range` from the request query, and craft the appropriate URL for the IEX API. We'll then make a request to that URL using `request`, and forward the response:

```js
// server.js

// ...

app.get("/", (req, res) => {
  // Read query parameters.
  const symbol = req.query["symbol"];
  const range = req.query["range"];
  // Craft IEX API URL.
  const url = `https://api.iextrading.com/1.0/stock/market/batch?symbols=${symbol}&types=quote,chart&range=${range}`;
  // Make request to IEX API and forward response.
  request(url).pipe(res);
});
```

### Dealing with CORS

If we start our server now, and point our React client to it, we'll still see CORS errors. That's because our server isn't emitting CORS headers yet. Even though there are packages to help us deal with CORS, our needs are so basic that we can emit the necessary header ourselves with this simple middleware:

```js
// server.js

// ...

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  next();
});
```

Now all responses will have an `Access-Control-Allow-Origin` header set to `*`, so we'll allow requests from all origins.

### Deploying to Heroku

In order to deploy this app to [Heroku](https://www.heroku.com/) we'll want to make two small changes:

The first one is regarding the port our server listens to. We had hardcoded it to be `3001`, but if you read the docs for [Deploying Node.js Apps on Heroku](https://devcenter.heroku.com/articles/deploying-nodejs#specifying-a-start-script) you'll see this bit:

> The command in a web process type must bind to the port number specified in the `PORT` environment variable. If it does not, the dyno will not start.

So our `port` declaration should be changed to this:

```js
const port = process.env.PORT || 3001;
```

When we're running inside Heroku, we'll use whatever value the environment variable is set to. When we're running the app locally, that environment variable won't be set, so we'll fall back to port `3001`.

The other small change we'll want to make is to limit which origins can make requests to our server. We've been allowing `*` until now, but in production we may want to do this:

```js
res.header("Access-Control-Allow-Origin", process.env.ORIGIN || "*");
```

When running inside Heroku, we can set the `ORIGIN` environment variable to something like `https://groundberry.github.io` so that only our GitHub projects can access it. When we're running the app locally, the environment variable won't be set, so we'll fall back to `*` to allow all origins.

That's it, we're ready to deploy the app to Heroku:

```
$ heroku login
$ heroku create stock-chart-proxy
$ git push heroku master
...
-----> Node.js app detected
...
-----> Launching...
       Released v9 
       https://stock-chart-proxy.herokuapp.com/ deployed to Heroku
```

If we now point our React app to `https://stock-chart-proxy.herokuapp.com/`, we'll see requests like `https://stock-chart-proxy.herokuapp.com/?symbol=PRGS&range=5y` going through without any CORS issues, and everything will work as expected. Hooray!
