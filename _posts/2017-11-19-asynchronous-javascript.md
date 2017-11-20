---
layout: post
title:  "Asynchronous JavaScript"
date:   2017-11-19
categories: development
---

When we execute a function synchronously, we wait for it to finish before moving on to the next function.

```js
console.log('1');
console.log('2');
console.log('3');

// 1
// 2
// 3
```

When we execute a function asynchronously, we can move on to another function before the first one finishes.

```js
console.log('1');
setTimeout(() => { console.log('2') }, 100);
console.log('3');

// 1
// 3
// 2
```

The concept of asynchrony can be challenging for beginners but with practice it can become a very useful tool for developing efficient programs.

There are a few different ways of dealing with asynchrony in JavaScript. In this article we'll talk about callbacks, promises, events, and `async`/`await`. By the end of it you'll be able to decide which one is best for your particular situation.

## Callbacks

[Callbacks](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function) are the first option to run our code asynchronously. We pass a function as an argument to another function, so that it will be called at a later time:

```js
const fs = require('fs');

fs.readFile('/etc/passwd', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

In Node, most callbacks follow the convention of receiving an error as the first argument, and the result of the asynchronous operation as the second argument.

In our example, if the execution of `readFile` is not successful, we receive an error as the first argument and we throw it, so nothing gets printed to the console. If the execution of `readFile` is successful, the first argument will be `null`, so we will print the contents of the second argument to the console.

(You can learn more about `readFile` in the [Node.js documentation](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback).)

### Trick question

What do you think will be printed to the console after this code executes?

```js
for (var i = 3; i > 0; i--) {
  setTimeout(() => {
    console.log(i + ' seconds left!')
  }, (4 - i) * 1000);
}
```

<details>
  <summary>See answer here! 🔎</summary>
  <pre class="highlight"><code><span class="c1">// 0 seconds left!</span>
<span class="c1">// 0 seconds left!</span>
<span class="c1">// 0 seconds left!</span>
</code></pre>
</details>

We want to execute a function every second for three seconds. But we're seeing that, by the time the function is run, the loop has finished looping through the range, so we print `0 seconds left` three times. We can solve this by wrapping our call to `setTimeout` in a closure, so that each iteration preserves its own context:

```js
for (var i = 3; i > 0; i--) {
  (function (i) {
    setTimeout(() => {
      console.log(i + " seconds left!")
    }, (4 - i) * 1000);
  })(i);
}

// 3 seconds left!
// 2 seconds left!
// 1 seconds left!
```

We can also use `let` instead of `var` if we are working with ES6, making it a lot simpler and clean, since `let` declares variables that are limited in scope to the block:

```js
for (let i = 3; i > 0; i--) {
  setTimeout(() => {
    console.log(i + " seconds left!")
  }, (4 - i) * 1000);
}

// 3 seconds left!
// 2 seconds left!
// 1 seconds left!
```


## Promises

Another way of dealing with asynchrony in our APIs is through the use of [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

An asynchronous API that uses promises will return an instance of `Promise` when you call it. By calling its `then` method, we'll be able to provide a function that will get invoked whenever the asynchronous operation completes:

```js
const myImage = document.createElement('img');

fetch('flowers.jpg') // this returns a promise
  .then((response) => {
    return response.blob();
  });
```

We can return a new promise in our function, and chain multiple `then`s:

```js
fetch('flowers.jpg') // this returns a promise
  .then((response) => {
    return response.blob(); // this also returns a promise
  })
  .then((blob) => {
    const objectURL = URL.createObjectURL(blob);
    myImage.src = objectURL;
  });
```

We can also use the `catch` method of promises to handle errors. If any promises in the chain fail, the function we passed to `catch` will get called:

```js
fetch('flowers.jpg')
  .then((response) => {
    if (response.ok) {
      return response.blob();
    }
    throw new Error('Network response was not OK');
  })
  .then((blob) => {
    var objectURL = URL.createObjectURL(blob);
    myImage.src = objectURL;
  })
  .catch((error) => {
    console.error(error.message);
  });
```

(You can learn more about using `fetch` for your API requests in the [MDN website](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).)

### util.promisify

`util.promisify` is a Node utility function that converts a callback-based function into a promise-based one.

```js
const fs = require('fs');
const util = require('util');

const readFilePromise = util.promisify(fs.readFile);

readFilePromise('/etc/passwd')
  .then((data) => {
    console.log(data);
  })
  .catch((err) => {
    throw err;
  });
```

(You can learn more about `promisify` in the [Node.js documentation](https://nodejs.org/api/util.html#util_util_promisify_original).)

## Events

Callbacks and promises are a bit limited, as they only inform us when the operation has completed or failed, and only run once. What if we need to be notified of the progress of an operation? Events give us that extra flexibility.

In the following example we'll read a file. If something goes wrong, an `error` event will be triggered, and our first function will execute throwing an error message. As the file is read, one or many `data` events will be triggered (depending on file size), and our second function will store the different chunks. Once the file has been read completely, a `close` event will be triggered, and our third function will join all chunks and print them to the console.

```js
const fs = require('fs');

const readStream = fs.createReadStream('/etc/passwd');
const chunks = [];

readStream.on('error', err => {
  throw err;
});

readStream.on('data', chunk => {
  console.log('Reading...')
  chunks.push(chunk);
});

readStream.on('close', () => {
  console.log(chunks.join(''));
});
```

This is great, since it allows us to show a progress bar while the file is being read, so that the user receives feedback while the function executes.

## async/await

`async`/`await` is a JavaScript feature built on top of promises. If we have a block of code that uses promises we can make it look synchronous by using `async`/`await`.

First, we wrap our code in an [`async` function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function). Inside this `async` function we can use the [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator to stop the execution of our code until a promise is resolved.

> The purpose of `async`/`await` functions is to simplify the behavior of using promises synchronously and to perform some behavior on a group of `Promise`s. Just as `Promise`s are similar to structured callbacks, `async`/`await` is similar to combining generators and promises.

We can write the same function we did with promises but using `async`/`await`.

```js
async function loadImage(img, src) {
  try {
    const response = await fetch(src);
    const blob = await response.blob();
    const objectURL = URL.createObjectURL(blob);
    img.src = objectURL;
  }
  catch (err) {
    console.error(error.message);
  }
}

const myImage = document.createElement('img');
loadImage(myImage, 'flowers.jpg');
```

Now you know the different ways of dealing with asynchrony in JavaScript, and their benefits. You have the power to choose the one that better fits your needs, and write clean, readable, and maintainable code! 👩‍💻
