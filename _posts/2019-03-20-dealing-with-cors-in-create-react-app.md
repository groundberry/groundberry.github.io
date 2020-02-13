---
layout: post
title: "Dealing with CORS in Create React App"
date: 2019-03-20
categories: development
canonical: https://www.telerik.com/blogs/dealing-with-cors-in-create-react-app
---

If you've ever built a web app that had to request data from a different domain, you probably had to wrap your head around the browser's [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) and [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

In this article we'll learn how to get around CORS issues using [Create React App](https://facebook.github.io/create-react-app/)'s proxying capabilities.

## The problem

If our app is hosted under a certain domain (e.g. `domain1.com`), and it tries to make a request to an API that lives under a different domain (e.g. `domain2.com`) then the browser's same-origin policy kicks in, and blocks the request.

CORS is a feature that allows `domain2.com` to tell the browser that it's cool for `domain1.com` to make requests to it, by sending certain HTTP headers.

However, CORS can be tricky to get right, so sometimes people avoid it altogether by serving their frontend and backend under the same domain in production.

Create React App allows us to replicate this setup in development, so that we don't have to deal with CORS there either. It provides two options to do so: one that's very straightforward but is not very flexible, and one that requires a bit more work but is very flexible.

## Automatic proxying

We can tell Create React App to intercept requests to unknown routes and send them to a different domain, using the `proxy` option in `package.json`. It looks something like this:

```json
{
  "name": "flickr-client",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.8.6",
    "react-dom": "^16.8.6",
    "react-scripts": "^2.1.8"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "proxy": "http://localhost:4000"
}
```

When we start our app, it will be served under `http://localhost:3000`. If we request the root path `/`, then Create React App will respond with the corresponding HTML for our app. But if we were to request a different path like `/api`, Create React App would transparently forward it to `http://localhost:4000/api`.

If we look at the networks requests in your browser dev tools, we'll see that the request was made to `http://localhost:3000/api`, but it was in fact served by `http://localhost:4000/api` without the browser knowing.

It can't get easier than this!

### Manual proxying

If we need more control over how these cross-domain requests get made, we have another option, which is to create a file `src/setupProxy.js` that looks like this:

```js
module.exports = function(app) {
  // ...
};
```

That function receives `app`, an instance of an [Express](https://expressjs.com/) app, so we can do whatever we want with it.

For example, we can use a middleware like [`http-proxy-middleware`](https://github.com/chimurai/http-proxy-middleware) to proxy requests just like we did with the `proxy` option:

```js
const proxy = require("http-proxy-middleware");

module.exports = app => {
  app.use(
    "/api",
    proxy({
      target: "http://localhost:4000",
      changeOrigin: true
    })
  );
};
```

But we can go further, and use `http-proxy-middleware`'s options like `pathRewrite` to change the path of the request:

```js
const proxy = require("http-proxy-middleware");

module.exports = app => {
  app.use(
    "/api",
    proxy({
      target: "http://localhost:4000",
      changeOrigin: true,
      pathRewrite: {
        "^/api": "/api/v1"
      }
    })
  );
};
```

With this configuration, a request made to `http://localhost:3000/api/foo` will be forwarded to `http://localhost:4000/api/v1/foo`.

We could also add a logger like [`morgan`](https://github.com/expressjs/morgan) while we're at it:


```js
const proxy = require("http-proxy-middleware");
const morgan = require("morgan");

module.exports = app => {
  app.use(
    "/api",
    proxy({
      target: "http://localhost:4000",
      changeOrigin: true,
      pathRewrite: {
        "^/api": "/api/v1"
      }
    })
  );

  app.use(morgan('combined'));
};
```

So now every time a request gets made to our proxy, it will get logged to the console.

The possibilities are truly endless.

## Conclusion

If your web app needs to request data from a different domain, and you want your development environment to mimic a production configuration where frontend and backend are served from the same domain, make sure to take a look at the `proxy` and `src/setupProxy.js` options of Create React App. They'll make development of your app much easier!
