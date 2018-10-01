---
layout: post
title:  "5 things I didn't know about Create React App"
date:   2018-09-29
categories: development
canonical: https://www.telerik.com/blogs/5-things-i-didnt-know-about-create-react-app
---

[Create React App](https://github.com/facebook/create-react-app) is a tool that makes it really easy to create React apps without having to deal with complex configurations. The recent release of Create React App v2 is a great excuse to go through their [User Guide](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md) one more time, and find interesting features I didn't know about. Here are my highlights.

## 1. Displaying lint errors in the editor

I love linters! They help me identify potential problems as I write my code, before I even get the chance to run it. Create React App already comes with [ESLint](https://eslint.org/) installed, and with some rules configured by default, but it only displays linting warnings and errors in the terminal:

{% include image.html name="terminal_error.png" alt="ESLint error in the terminal" %}

What I really want is to see those warnings and errors directly in my editor, so that I can fix them immediately without having to switch contexts.

It turns out Create React App makes it as easy as adding a `.eslintrc` file at the root of the project with this content:

```json
{
  "extends": "react-app"
}
```

If you have your editor properly configured (I use the [ESLint extension for VSCode](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)), then you'll see the results immediately:

{% include image.html name="editor_error.png" alt="ESLint error in the editor" %}

## 2. Formatting code automatically using Prettier

[Prettier](https://prettier.io/) is an opinionated code formatter that enforces a consistent style in all our files. I've started using it in all my projects, because it allows me to concentrate on the code itself, and forget about formatting.

We can run it from the command line (install it with`npm install --global prettier`, and then run `prettier` in our project) or from our editor (I use the [Prettier extension for VSCode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)). But another popular way of running Prettier is via Git hooks.

If you've never heard of hooks, they are scripts that Git runs when certain actions happen. For example, a pre-commit hook runs every time we execute `git commit`, before the commit itself is created. We can invoke Prettier from a pre-commit hook to format all our staged files, and ensure that everything we commit to our repo is properly formatted.

While we could write that hook by hand (take a look at your `.git/hooks` folder to check out some examples), there are two npm modules that help us with the process, [`husky`](https://github.com/typicode/husky) and [`lint-staged`](https://github.com/okonet/lint-staged), and they integrate perfectly fine with Create React App.

Let's install Prettier and those two modules:

```
npm install --save-dev prettier husky lint-staged 
```

Then we'll add the following sections to the end of the `package.json` file in our app:

```json
{
  // ...
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,json,css}": [
      "prettier --write",
      "git add"
    ]
  }
}
```

Now every time we commit, we'll see `husky` invoke `lint-staged`, which will in turn invoke `prettier` on all the files we're about to commit.

{% include video.html name="prettier_in_action.mp4" alt="Prettier in action" %}

Neat, huh?

## 3. Developing components in isolation

If we are working on a complex app with many components and different states for each component, every time we make a change we have to reload the whole app and interact with it until we get it to the desired state.

A different way of working is to use tools such as [Storybook](https://storybook.js.org/) and [Styleguidist](https://react-styleguidist.js.org/), which allow us to develop each component in isolation.

I'm particularly fond of Storybook, because integrating it with Create React App is such a breeze:

```
npm install --global @storybook/cli
getstorybook
```

{% include image.html name="installing_storybook.png" alt="Installing Storybook" %}

After the wizard finishes doing its job, we just need to run `npm run storybook` and start writing stories for our components in the `stories/` folder that the wizard created.

We can add a new story for our `Header` component like this:

```js
import React from 'react';
import { storiesOf } from '@storybook/react';
import Header from '../Header';

storiesOf('Header', module)
  .add('default theme', () => <Header />)
  .add('light theme', () => <Header theme="light" />)
  .add('dark theme', () => <Header theme="dark" />);
```

Which will create a new section named `Header` in our storybook:

{% include image.html name="storybook_stories.png" alt="Adding new stories to Storybook" %}

Then we can continue developing it from there!

## 4. Making a Progressive Web App

The only requirements for our app to be considered a PWA are:

1. It must be served over HTTPS.
2. It must provide a manifest.
3. It must register a `ServiceWorker`.

We're probably already serving your app over HTTPS, so the only things left to do are the manifest and the `ServiceWorker`.

Luckily, Create React App already generates a manifest for us, located at `public/manifest.json`. We'll just need to tweak its values.

It also generates a `ServiceWorker`, but doesn't register it by default [for reasons outlined in their User Guide](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#why-opt-in). After reading that section and understanding their reasoning, if you want to go ahead, open `src/index.js` and look for the following:

```js
// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: http://bit.ly/CRA-PWA
serviceWorker.unregister();
```

Now turn `serviceWorker.unregister()` into `serviceWorker.register()` and you're done. You have a PWA, and Chrome will offer your users to add it to their homescreen!

{% include image.html name="pwa_homescreen.png" alt="Chrome offers to add the app to your homescreen" %}

## 5. Code splitting

Code splitting is a feature of modern JavaScript bundlers that allows us to split our app into smaller chunks which can then be loaded on demand.

Create React App v2 supports code splitting via dynamic `import()` statements. That is, if it encounters a call to `import('./someModule')` when building your app, it will create a new chunk for `someModule` and all its dependencies, totally separate from our entry bundle.

Let's see that with an example. Imagine we have a complex form that is only displayed when the user clicks a button. We can use code splitting to avoid downloading, parsing, and executing all that code on page load, and instead wait to load the form until the user clicks said button.

Here's our complex form using [`formik`](https://github.com/jaredpalmer/formik) and [`yup`](https://github.com/jquense/yup):

```js
import React, { Component } from "react";
import { Formik } from "formik";
import * as Yup from "yup";

const formValidator = Yup.object().shape({ /* ... */ });

export default class Form extends Component {
  render() {
    return (
      <Formik validationSchema={formValidator}>
        {/* ... */}
      </Formik>
    );
  }
}
```

And here's our app using dynamic `import()` to load the form on demand:

```js
import React, { Component } from "react";

export default class App extends Component {
  constructor() {
    super();

    this.state = {
      Form: undefined
    };
  }

  render() {
    const { Form } = this.state;

    return (
      <div className="app">
        {Form ? <Form /> : <button onClick={this.showForm}>Show form</button>}
      </div>
    );
  }

  showForm = async () => {
    const { default: Form } = await import("./Form");
    this.setState({ Form });
  };
}
```

It's only when the user clicks the button that we incur in the cost of loading `Form`. Once the `import()` promise resolves, we call `setState` and force a re-render of the app with the loaded component.

{% include video.html name="code_splitting.mp4" alt="Code splitting in our app" %}

If we look closely at the network requests being made, we'll notice two new chunks (`0.chunk.js` and `1.chunk.js`) being requested after we click the button. They contain `Form` and its dependencies `formik` and `yup`, so we managed to avoid downloading all that code on initial page load, making our app feel faster!

## Wrapping up

Create React App is a wonderful tool that makes it very easy to get started with React. It also contains a ton of features, so it pays to read its documentation in order to get all its benefits.
