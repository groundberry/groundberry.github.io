---
layout: post
title:  "The new way of profiling React apps"
date:   2019-03-03
categories: development
published: false
---

In my previous article [Profiling React components with the User Timing API]({% post_url 2018-09-06-profiling-react-components-with-the-user-timing-api %}), we talked about measuring the rendering times of our components using the browser's profiling API and tooling. Shortly after I wrote that article, the React team announced [a brand new `Profiler` API, and a *Profiler* tab in DevTools](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html). While the `Profiler` API is still flagged as unstable as of version `16.8`, it's the recommended way of measuring the rendering times of our components, because it'll be fully compatible with features such as [time slicing and suspense](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html).

In this article we'll measure the performance of [an example React app](https://github.com/groundberry/stock-chart-client) with both the *Profiler* tab in DevTools, and the `Profiler` component.

## Using the Profiler tab from React DevTools

If we are working on our React app in development mode, we can use the *Profiler* tab in React DevTools to record parts of its execution, and then analyze all the updates that React made. (If we want to use the *Profiler* tab on a production app, we need to make [some changes to our config](https://gist.github.com/bvaughn/25e6233aeb1b4f0cdb8d8366e54a3977).)

To profile our app, we just need to switch to the *Profiler* tab, and press the *Record* button to start profiling:

{% include image.html name="profiling-app-start.png" alt="Start profiling by pressing the Record button" %}

We'll then perform actions on our app, and press the *Record* button again to stop profiling. The DevTools will show us each of the updates that happened while we were recording, using a fancy flame chart:

{% include image.html name="profiling-app-end.png" alt="Stop profiling by pressing the Record button again" %}

If you are not familiar with this way of representing performance data, you may be wondering what all these colored bars mean. Let's break it down.

Every time any of our components *render*, React compares the resulting tree of components with the current one. If there are changes, React will take care of applying them to the DOM in a phase called *commit*.

The colored bars we're seeing at the top are commits that happened while we were recording. The yellow/orange bars are the ones with higher rendering times, so we should probably pay extra attention to them:

{% include image.html name="commits.png" alt="Commits made while we were recording" %}

If we click on one of those commits, the flame chart below will be updated, showing the components that changed in that commit as horizontal bars. The longer the bar, the more time it took for that component to render:

{% include image.html name="flame-chart.png" alt="Flame chart of component rendering times" %}

The chart shows the root component at the top, with its children sitting below in hierarchical order. The number shown inside each bar represents the time it took to render the component and its children. When we see something like **RangeButtons (0.3ms of 2.4ms)**, it means that `RangeButtons` took 0.3ms to render, while `RangeButtons` plus its only child `ButtonGroup` took 2.4ms. That means `ButtonGroup` must have taken ~2ms to render, which is confirmed when we look at the bar below that says **ButtonGroup (0.5ms of 2.0ms)**.

Another cool thing we can do here is click on the bar for a certain component. Not only will the flame chart focus on the selected component, but the pane on the right will also show us how many times it has rendered for the lifetime of the app, and its props and state for the current commit:

{% include image.html name="component.png" alt="StockChart app commit times" %}

The *Profiler* tab in React DevTools is a great way of inspecting how our app is performing without needing to change our code. Just by recording key interactions, we'll be able to know where rendering time is going, and identify bottlenecks that make our app sluggish.

## Using the `Profiler` component

If we want to have programmatic access to the performance measurements of a specific component, we can use the new `Profiler` component. It works very similarly to the `MeasureRender` component we built in our previous post, by wrapping part or all of our app tree, and giving us metrics on how long it took for that tree to render.

The first thing we have to do to use the `Profiler` component is to import it. It's still flagged as unstable at the moment of writing, but we'll rename the import to get nicer looking code:

``` js
import React, { unstable_Profiler as Profiler } from "react";
```

The `Profiler` component can then be used to wrap any part of our tree of components:

```js
// CustomStockChart.js

const CustomStockChart = props => {
   // ...

  return (
    <Profiler id="StockChart" onRender={logTimes}>
      <StockChart>
        {/* ... */}
      </StockChart>
    </Profiler>
  );
};

const logTimes = (id, phase, actualTime, baseTime, startTime, commitTime) => {
  console.log(`${id}'s ${phase} phase:`);
  console.log(`Actual time: ${actualTime}`);
  console.log(`Base time: ${baseTime}`);
  console.log(`Start time: ${startTime}`);
  console.log(`Commit time: ${commitTime}`);
};

export default CustomStockChart;
```

When `CustomStockChart` renders, the `Profiler`'s `onRender` callback will be invoked with a bunch of useful information. In our example, it'll print something like this to the console:

```
StockChart's mount phase:
Actual time: 7.499999995867256
Base time: 7.1249999981955625
Start time: 384888.51500000054
Commit time: 384897.5449999998

StockChart's update phase:
Actual time: 0.3500000038766302
Base time: 7.075000001175795
Start time: 385115.2050000001
Commit time: 385116.22499999974
```

The meaning of each of these arguments is explained in this [awesome gist by Brian Vaughn](https://gist.github.com/bvaughn/60a883af01716a03a1b3285a1029be0c). In the real world, instead of logging them to the console, you would probably be sending them to your backend in order to get useful aggregate charts, like we suggested in our previous post.

Anyways, be sure to spend time understanding these two new tools in your arsenal, as they'll prove invaluable when trying to identify performance issues in your React apps!