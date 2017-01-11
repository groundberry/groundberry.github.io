---
layout: post
title:  "Higher-order functions in JavaScript"
date:   2017-01-11
categories: development
---

In JavaScript, functions are [first-class objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions). This means that they are like *first-class citizens* in the JavaScript world. They are treated as values, and can do the same things that numbers or strings can, such as being passed as arguments to other functions, stored in data structures, or returned.

When functions receive other functions as arguments, or return another function, they are called [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function). Examples of higher-order functions are `Array.prototype.filter()`, `Array.prototype.map()`, and `Array.prototype.reduce()`.

## Filter

[`Array.prototype.filter()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) returns a new array with the elements that satisfy the condition we set in the function we are passing. Let's check it out with this example of healthy food. 😁

```js
var dishes = [
  { name: 'apple pie', type: 'fruit', time: 25 },
  { name: 'spinach and ricotta tortellini', type: 'vegetable', time: 55 },
  { name: 'banana split', type: 'fruit', time: 15 },
  { name: 'apple strudel', type: 'fruit', time: 45 },
];

var fruityDishes = dishes.filter(function(dish) {
  return dish.type === 'fruit';
});
// [
//   {name: 'apple pie', ...},
//   {name: 'banana split', ...},
//   {name: 'apple strudel', ...}
// ]
```

We can refactor this by storing the function as a variable that we can reuse many times, and pass the name of the variable to `filter()` as an argument:

```js
var isFruityDish = function(dish) {
  return dish.type === 'fruit';
};

var fruityDishes = dishes.filter(isFruityDish);
// [
//   {name: 'apple pie', ...},
//   {name: 'banana split', ...},
//   {name: 'apple strudel', ...}
// ]
```

Nice!

## Map

[`Array.prototype.map()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/map) returns a new array with the result of calling a function on every element of the original array. In this case we are returning an array with the names of the recipes made with fruit.

```js
var dishes = [
  { name: 'apple pie', type: 'fruit', time: 25 },
  { name: 'spinach and ricotta tortellini', type: 'vegetable', time: 55 },
  { name: 'banana split', type: 'fruit', time: 15 },
  { name: 'apple strudel', type: 'fruit', time: 45 },
];

var fruityNames = dishes.filter(function(dish) {
  return dish.type === 'fruit';
}).map(function(dish) {
  return dish.name;
});
// ['apple pie', 'banana split', 'apple strudel']  
```

The refactored version of this would be something like:

```js
var isFruityDish = function(dish) {
  return dish.type === 'fruit';
};

var dishName = function(dish) {
  return dish.name;
};

var fruityNames = dishes.filter(isFruityDish).map(dishName);
// ['apple pie', 'banana split', 'apple strudel']    
```

## Reduce

[`Array.prototype.reduce()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) reduces all the elements in an array to a single value, applying a function to every element (`cur`) of the original array from left (`dishes[0]`) to right (`dishes[dishes.length - 1]`), and accumulating the result as it goes (`prev`). In this example we are adding the time needed to cook the selected recipes and returning the total.

```js
var dishes = [
  { name: 'apple pie', type: 'fruit', time: 25 },
  { name: 'spinach and ricotta tortellini', type: 'vegetable', time: 55 },
  { name: 'banana split', type: 'fruit', time: 15 },
  { name: 'apple strudel', type: 'fruit', time: 45 },
];

var totalTime = dishes.filter(function(dish) {
  return dish.type === 'fruit';
}).map(function(dish) {
  return dish.time;
}).reduce(function(prev, cur) {
  return prev + cur;
}, 0);
// 85
```

The refactored version of this would be something like:

```js
var isFruityDish = function(dish) {
  return dish.type === 'fruit';
};

var dishTime = function(dish) {
  return dish.time;
};

var sum = function(prev, cur) {
  return prev + cur;
};

var totalTime = dishes
  .filter(isFruityDish)
  .map(dishTime)
  .reduce(sum, 0);
// 85
```

## Cooking our own

Let's write our own higher-order function that cooks a dish in a given cooking appliance:

```js
var cook = function(appliance) {
  return function(dish) {
    console.log('Cooking ' + dish + ' in ' + appliance + '...');
  };
};

var cookInOven = cook('oven');
cookInOven('apple strudel');
// Cooking apple strudel in oven...
cookInOven('apple pie');
// Cooking apple pie in oven...
```

Notice that the function returned when we call the `cook()` function remembers its context. Every time we call the `cookInOven()` function, passing a different dish as an argument, it will remember that they are being cooked in an `oven`. This is what is known as a [closure](https://developer.mozilla.org/en/docs/Web/JavaScript/Closures).

Now that we know a bit more about the power of functional programming, we can use it to create more maintainable and clean code. Cool! 😎
