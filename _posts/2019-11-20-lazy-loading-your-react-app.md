---
layout: post
title: "Lazy-loading your React app"
date: 2019-11-20
categories: development
published: false
---

If you're building a web app today, chances are you're using a JavaScript framework like React, together with a bunch of other libraries like React Router or Kendo UI. We often forget to consider the cost of sending all this JavaScript to our users. According to Google's V8 team in their report [The cost of JavaScript 2019](https://v8.dev/blog/cost-of-javascript-2019), anywhere up to 30% of a page's load time is spent in JavaScript execution.

> JavaScript is still the most expensive resource we send to mobile phones, because it can delay interactivity in large ways.
>
> -- Addy Osmani

In this article we are going to discuss how we can improve the performance of our apps by loading only the JavaScript that the user needs at any point in time, reducing the amount of code they have to download and execute on page load, and making the app interactive faster.

We'll use [`React.lazy` and `Suspense`](https://reactjs.org/docs/code-splitting.html) to delay the loading of a complex component like [KendoReact's `StockChart`](https://www.telerik.com/kendo-react-ui/components/charts/stockchart/) until a button is clicked.

{% include image.html name="sample-app.png" alt="Sample app using React.lazy and Suspense" %}

You can see the code for the app in this [GitHub repository](https://github.com/groundberry/lazy-app).

## Understanding dynamic imports

Instead of sending a big bundle with all the code for our app on initial page load, we can send smaller bundles gradually as the user interacts with the app. To do this we'll rely on a modern JavaScript feature called [dynamic imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports). A dynamic import returns a promise that will resolve once the required module gets transferred over the network, and is parsed and executed by the JavaScript engine.

A static import looks like this:

```js
import { concat } from "./utils";

console.log(concat("A", "B", "C"));
```

While a dynamic import looks like this:

```js
import("./utils").then(utils => {
  console.log(utils.concat("A", "B", "C"));
});
```

Tools like [Create React App](https://create-react-app.dev/) and [Webpack](https://webpack.js.org/) understand what we're trying to do with these dynamic imports, and will output separate JavaScript files for these lazy-loaded bundles. If we're configuring Webpack ourselves, it may be a good idea to spend some time reading [Webpack's documentation on code-splitting](https://webpack.js.org/guides/code-splitting/).

## Lazy-loading with `React.lazy` and `Suspense`

Starting with version 16.6, React includes a built-in [`React.lazy`](https://reactjs.org/docs/code-splitting.html#reactlazy) function that makes it very easy to split an application into lazy-loaded components using dynamic imports.

You can turn this:

```js
import StockChartContainer from "./StockChartContainer";
```

Into this:

```js
const StockChartContainer = lazy(() => import("./StockChartContainer"));
```

And React will automatically load the bundle containing our `StockChartContainer` component when we try to render it for the first time.

We'll want to wrap this lazy component inside a `Suspense` component, which will allow us to show some fallback content while things are being loaded. Let's see what that looks like.

## Example

In this example we are going to be loading a complex component containing [KendoReact's `StockChart`](https://www.telerik.com/kendo-react-ui/components/charts/stockchart/), but only after the user clicks on a button. This way we'll avoid sending the user more code than they need on initial load.

We'll store state to track whether our complex component needs to be displayed:

```js
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      showChart: false
    };
  }
}
```

Then, we'll implement a `handleClick` function that will toggle state when the user clicks a button:

```js
class App extends Component {
  // ...

  handleClick = () => {
    this.setState(prevState => ({
      showChart: !prevState.showChart
    }));
  };
}
```

Now we just need to put it all together in the `render` method:

```js
const StockChartContainer = lazy(() => import("./StockChartContainer"));

class App extends Component {
  // ...

  render() {
    const { showChart } = this.state;
    const buttonText = showChart ? "Hide Stock Chart" : "Show Stock Chart";
    const chartComponent = showChart ? <StockChartContainer /> : null;
    const loadingComponent = <div>Loading...</div>;

    return (
      <div className="App">
        <header className="App-header">
          <h1 className="App-title">Stock Chart</h1>
          <div className="App-button">
            <Button primary={true} onClick={this.handleClick}>
              {buttonText}
            </Button>
          </div>
        </header>
        <div className="App-chart">
          <Suspense fallback={loadingComponent}>{chartComponent}</Suspense>
        </div>
      </div>
    );
  }
}
```

Let's see if it worked. If we open Chrome Dev Tools, click on the *Network* tab, and reload the page, we'll see the bundles we send on initial load:

{% include image.html name="bundles-initial.png" alt="Bundles on initial load" %}

If we now click on the "Show Stock Chart" button, we'll see that more bundles get transferred right before our chart gets displayed:

{% include image.html name="bundles-lazy.png" alt="Lazy-loaded bundles" %}

We were able to delay the download and execution of all that code until the user needed it. Awesome!

## Conclusion

If we send too much JavaScript to our users, we'll make the browser's main thread busy, and it won't be able to respond to user interaction. Lazy-loading components of our app that are not needed on initial page load will help reduce the amount of work the browser has to do, which will drive down our time-to-interactive, and provide a better experience to our users, especially those on mobile devices. `React.lazy` and `Suspense` make it so easy to do that we really have no excuse!
