---
layout: post
title: "Internationalizing a React app with KendoReact"
date: 2018-11-15
categories: development
published: false
---

When we're building web applications, we are doing so with the intent of attracting traffic, and turning visitors into paying customers. If someone visiting our app can't make sense of its content because they don't speak our language, or they can't complete our payment form because their currency is not supported, they're going to walk away, and we're going to lose that customer.

If we internationalize our app, we design and develop it in a way that ensures it will work well for users from any culture, region, or language, so we maximize its reach.

In this article we'll build an [example React app fully internationalized](https://groundberry.github.io/example-kendo-react-intl/), formatting dates, numbers and currencies, and translating messages, using the [KendoReact Internationalization API](https://www.telerik.com/kendo-react-ui/components/globalization/i18n/):

{% include video.html name="internationalized_app.mp4" alt="Our internationalized app switching from English to Spanish locale" %}

## Getting started

If you don't have an existing React app to follow along, the easiest way to create one is to use [Create React App](https://github.com/facebook/create-react-app):

```
npx create-react-app my-app
cd my-app
npm start
```

Now that we have a React app to internationalize, we'll install the `kendo-react-intl` package, which works perfectly fine with both Create React App and with custom React apps:

```
npm install --save @progress/kendo-react-intl
```

We'll also need to load portions of the [Common Locale Data Repository](http://cldr.unicode.org/). The CLDR is a huge repository of locale-specific rules and patterns, for things like capitalization, pluralization, formatting and parsing dates, times, numbers and currency values, among other things.

We only require these three CLDR packages:

```
npm install --save cldr-core cldr-dates-full cldr-numbers-full
```

## Loading CLDR data

The first thing we'll need to do is import the CLDR data and let `kendo-react-intl` know about it. For now, we'll only load the data associated to the English locale. In our `index.js` file we could do:

```js
// Common CLDR data.
import likelySubtags from "cldr-core/supplemental/likelySubtags.json";
import weekData from "cldr-core/supplemental/weekData.json";
import currencyData from "cldr-core/supplemental/currencyData.json";
// CLDR data for English locale.
import enCalendar from "cldr-dates-full/main/en/ca-gregorian.json";
import enDateFields from "cldr-dates-full/main/en/dateFields.json";
import enNumbers from "cldr-numbers-full/main/en/numbers.json";
import enCurrency from "cldr-numbers-full/main/en/currencies.json";

import { load } from "@progress/kendo-react-intl";

// ...

load(
  likelySubtags,
  weekData,
  currencyData,
  enCalendar,
  enDateFields
  enNumbers,
  enCurrency
);

ReactDOM.render(<App />, document.getElementById("root"));
```

## Loading messages

We'll also need to let `kendo-react-intl` know about our translated messages for each locale. For this app, we'll be storing our messages in JSON format, with one file per locale, like this:

```json
{
  "product-start-date": "Start date",
  "product-seats-available": "Seats available",
  "product-price": "Price",
  "add-to-cart": "Add to cart"
}
```

Again, we'll only be loading the English translations for now:

```js
// English translations.
import enMessages from "./i18n/en.json";

import { loadMessages } from "@progress/kendo-react-intl";

// ...

loadMessages(enMessages, "en");

ReactDOM.render(<App />, document.getElementById("root"));
```

## Formatting dates, numbers and currencies

Now that we have our CLDR data and messages loaded, we can start internationalizing our app. We'll need to wrap our app in an [`IntlProvider`](https://www.telerik.com/kendo-react-ui/components/globalization/api/IntlProvider/), so that our components have access to the [`IntlService`](https://www.telerik.com/kendo-react-ui/components/globalization/api/IntlService/) that will allow us to format dates, numbers and currencies:

```js
import React, { Component } from "react";
import { IntlProvider } from "@progress/kendo-react-intl";

export default class App extends Component {
  render() {
    return (
      <IntlProvider locale="en">
        {/* ... */}
      </IntlProvider>
    );
  }
}
```

### Formatting dates

We can write a date formatting component relatively easily, using the `provideIntlService` and `registerForIntl` functions from `kendo-react-intl`, and the `formatDate` method that `IntlService` exposes:

```js
import { Component } from "react";
import {
  registerForIntl,
  provideIntlService
} from "@progress/kendo-react-intl";

class DateFormatter extends Component {
  render() {
    const { value } = this.props;

    return provideIntlService(this).formatDate(value, {
      date: "medium"
    });
  }
}

registerForIntl(DateFormatter);

export default DateFormatter;
```

### Formatting numbers

We can build a number formatting component following the same approach, and using the `formatNumber` method from `IntlService`:

```js
import { Component } from "react";
import {
  registerForIntl,
  provideIntlService
} from "@progress/kendo-react-intl";

class NumberFormatter extends Component {
  render() {
    const { value } = this.props;

    return provideIntlService(this).formatNumber(value);
  }
}

registerForIntl(NumberFormatter);

export default NumberFormatter;
```

### Formatting currencies

The `formatNumber` method from `IntlService` also allows us to format currencies, so we can build a currency formatting component like this:

```js
import { Component } from "react";
import {
  registerForIntl,
  provideIntlService
} from "@progress/kendo-react-intl";

class CurrencyFormatter extends Component {
  render() {
    const { value, currency } = this.props;

    return provideIntlService(this).formatNumber(value, {
      style: "currency",
      currency
    });
  }
}

registerForIntl(CurrencyFormatter);

export default CurrencyFormatter;
```

## Translating messages

Just like we did with the `IntlProvider`, we'll wrap our app in a [`LocalizationProvider`](https://www.telerik.com/kendo-react-ui/components/globalization/api/LocalizationProvider/) so that our components have access to the `LocalizationService` that lets us load translated messages.

```js
import React, { Component } from "react";
import { IntlProvider } from "@progress/kendo-react-intl";

export default class App extends Component {
  render() {
    return (
      <IntlProvider locale="en">
        <LocalizationProvider language="en">
          {/* ... */}
        </LocalizationProvider>
      </IntlProvider>
    );
  }
}
```

We'll also write a component that displays a localized message, using the `provideLocalizationService` and `registerForLocalization` functions from `kendo-react-intl`, and the `toLanguageString` method that `LocalizationService` exposes:

```js
import { Component } from "react";
import {
  registerForLocalization,
  provideLocalizationService
} from "@progress/kendo-react-intl";

class LocalizedMessage extends Component {
  render() {
    const { message } = this.props;

    return provideLocalizationService(this).toLanguageString(message);
  }
}

registerForLocalization(LocalizedMessage);

export default LocalizedMessage;
```

## Putting it all together

Ok, we're ready to create an internationalized component! We'll be building a simple product card to sell our fictitious workshops:

```js
import React, { Component } from "react";
import CurrencyFormatter from "./CurrencyFormatter";
import DateFormatter from "./DateFormatter";
import LocalizedMessage from "./LocalizedMessage";
import NumberFormatter from "./NumberFormatter";

export default class Product extends Component {
  render() {
    const { date, number, price, currency } = this.props;

    return (
      <div>
        <p>
          <strong>
            <LocalizedMessage message="product-start-date" />
          </strong>
          : <DateFormatter value={date} />
        </p>
        <p>
          <strong>
            <LocalizedMessage message="product-seats-available" />
          </strong>
          : <NumberFormatter value={number} />
        </p>
        <p>
          <strong>
            <LocalizedMessage message="product-price" />
          </strong>
          : <CurrencyFormatter value={price} currency={currency} />
        </p>
        <div>
          <button>
            <LocalizedMessage message="add-to-cart" />
          </button>
        </div>
      </div>
    );
  }
}
```

And we'll render it from our `App`:

```js
import React, { Component } from "react";
import { IntlProvider, LocalizationProvider } from "@progress/kendo-react-intl";
import Product from "./Product";

class App extends Component {
  render() {
    return (
      <IntlProvider locale="en">
        <LocalizationProvider language="en">
          <Product
            date={new Date()}
            number={1000}
            price={99.99}
            currency="USD"
          />
        </LocalizationProvider>
      </IntlProvider>
    );
  }
}

export default App;
```

The result looks something like this:

{% include image.html name="internationalized_app.png" alt="Our internationalized app with proper date, number and currency formatting" %}

Notice how the date, number and currency are all properly formatted! 😍

## Switching locale and currency

Even though our app is internationalized, we have been using the English locale for everything. Let's change that, so that the user can choose their preferred locale!

We'll create a very straightforward component to select between two locales, English and Spanish, and between two currencies, dollars and euros:

```js
export default class Config extends Component {
  render() {
    const { locale, currency, onChangeLocale, onChangeCurrency } = this.props;

    return (
      <div className="Config">
        <label htmlFor="locale">Locale:</label>
        <select
          id="locale"
          name="locale"
          value={locale}
          onChange={onChangeLocale}
        >
          <option value="en">English</option>
          <option value="es">Spanish</option>
        </select>
        <label htmlFor="currency">Currency:</label>
        <select
          id="currency"
          name="currency"
          value={currency}
          onChange={onChangeCurrency}
        >
          <option value="USD">USD</option>
          <option value="EUR">EUR</option>
        </select>
      </div>
    );
  }
}
```

We'll add it to our `App` component, which will save these values as state:

```js
class App extends Component {
  constructor() {
    super();

    this.state = {
      locale: "en",
      currency: "USD"
    };
  }

  render() {
    const { locale, currency } = this.state;

    return (
      <IntlProvider locale={locale}>
        <LocalizationProvider language={locale}>
          <>
            <Product
              date={new Date()}
              number={1000}
              price={99.99}
              currency={currency}
            />
            <Config
              locale={locale}
              currency={currency}
              onChangeLocale={this.onChangeLocale}
              onChangeCurrency={this.onChangeCurrency}
            />
          </>
        </LocalizationProvider>
      </IntlProvider>
    );
  }

  onChangeLocale = event => {
    this.setState({
      locale: event.target.value
    });
  };

  onChangeCurrency = event => {
    this.setState({
      currency: event.target.value
    });
  };
}
```

If we run the app now and try switching to the Spanish locale, we'll encounter an error similar to this one:

{% include image.html name="no_locale_error.png" alt="Error shown when CLDR data for a locale is not yet loaded" %}

It's because we had only loaded CLDR data and messages for the English locale!

## Loading more locales naively

The naive way to solve this is to just import more stuff in our entry file `index.js`, and update our calls to `load` and `loadMessages`:

```js
// Common CLDR data.
import likelySubtags from "cldr-core/supplemental/likelySubtags.json";
import weekData from "cldr-core/supplemental/weekData.json";
import currencyData from "cldr-core/supplemental/currencyData.json";
// CLDR data for English locale.
import enCalendar from "cldr-dates-full/main/en/ca-gregorian.json";
import enDateFields from "cldr-dates-full/main/en/dateFields.json";
import enNumbers from "cldr-numbers-full/main/en/numbers.json";
import enCurrency from "cldr-numbers-full/main/en/currencies.json";
// CLDR data for Spanish locale.
import esCalendar from "cldr-dates-full/main/es/ca-gregorian.json";
import esDateFields from "cldr-dates-full/main/es/dateFields.json";
import esNumbers from "cldr-numbers-full/main/es/numbers.json";
import esCurrency from "cldr-numbers-full/main/es/currencies.json";
// ... more imports as you add support for more locales...

// English translations.
import enMessages from "./i18n/en.json";
// Spanish translations.
import esMessages from "./i18n/es.json";
// ... more translations as you add support for more locales...

import { load } from "@progress/kendo-react-intl";
import { loadMessages } from "@progress/kendo-react-intl";

load(
  likelySubtags,
  weekData,
  currencyData,
  enCalendar,
  enDateFields
  enNumbers,
  enCurrency,
  esCalendar,
  esDateFields
  esNumbers,
  esCurrency,
  // ...
);

loadMessages(enMessages, "en");
loadMessages(esMessages, "es");
// ...
```

This approach has a big downside though: our bundles are going to keep growing and growing as we add support for more locales! It is terribly wasteful, why would we load data for all these different locales upfront, when the user is only seeing one at a time?

Let's think of a better way.

## Loading more locales dynamically

If only we could import all those packages lazily when the user changes locale... Wait, that's what dynamic `import` statements are for! We can create a few helper functions that dynamically load the necessary packages for the specified locale:

```js
import { load } from "@progress/kendo-react-intl";
import { loadMessages } from "@progress/kendo-react-intl";

export const loadCldrDataForLocale = async locale => {
  const cldr = await Promise.all([
    import("cldr-core/supplemental/likelySubtags.json"),
    import("cldr-core/supplemental/currencyData.json"),
    import("cldr-core/supplemental/weekData.json"),
    import(`cldr-numbers-full/main/${locale}/numbers.json`),
    import(`cldr-numbers-full/main/${locale}/currencies.json`),
    import(`cldr-dates-full/main/${locale}/ca-gregorian.json`),
    import(`cldr-dates-full/main/${locale}/dateFields.json`)
  ]);
  load(...cldr);
};

export const loadMessagesForLocale = async locale => {
  const messages = await import(`./${locale}.json`);
  loadMessages(messages.default, locale);
};
```

We'll update our `App` to add one more piece of state, `loaded`, so that we can keep track of whether the CLDR data and messages for a specific locale have been loaded. If they haven't, we'll call our new helper functions to do so:

```js
export default class App extends Component {
  constructor() {
    super();

    this.state = {
      locale: "en",
      currency: "USD",
      loaded: false
    };
  }

  componentDidMount() {
    this.ensureLoaded();
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.locale !== this.state.locale) {
      this.ensureLoaded();
    }
  }

  render() {
    const { locale, currency, loaded } = this.state;

    if (!loaded) {
      return null;
    }

    return (
      <IntlProvider locale={locale}>
        <LocalizationProvider language={locale}>
          {/* ... */}
        </LocalizationProvider>
      </IntlProvider>
    );
  }

  ensureLoaded = async () => {
    const { locale, loaded } = this.state;

    if (loaded) {
      return;
    }

    await loadCldrDataForLocale(locale);
    await loadMessagesForLocale(locale);

    this.setState({
      loaded: true
    });
  };
}
```

If we check the network tab in Chrome dev tools, we'll see the CLDR data and messages being lazy-loaded when we switch locales:

{% include video.html name="lazy_loading_locales.mp4" alt="The network tab in Chrome dev tools showing our CLDR data and messages being lazily loaded" %}

You can find the source code for everything here: <https://github.com/groundberry/example-kendo-react-intl>

## Wrapping up

By internationalizing our app, we are helping it reach an audience as wide as possible. For legacy apps this can be a slow and costly process, but if you're starting from scratch the work is relatively straightforward, as we've seen, so there's no excuse not to do it!
