---
layout: post
title:  "On Cross-Site Scripting and Content Security Policy"
date:   2018-10-13
categories: development
canonical: https://www.telerik.com/blogs/on-cross-site-scripting-and-content-security-policy
---

October is [National Cyber Security Awareness Month](https://www.dhs.gov/national-cyber-security-awareness-month), so it's a great excuse to talk about one of the most common types of attacks to web applications, *Cross-Site Scripting*, and how to mitigate them using a security feature present in all modern browsers called *Content Security Policy*.

## Cross-Site Scripting

[Cross-Site Scripting](https://en.wikipedia.org/wiki/Cross-site_scripting), or XSS for short, is one of the most common security issues in web applications these days. In XSS, an attacker manages to inject their own code into our app. This code will then be executed when users visit the page, so it can do anything from stealing their cookies to logging all their key presses.

To demo this, we've built [a small app with a form that is vulnerable to XSS](https://stackblitz.com/edit/content-security-policies). The app works fine: we enter some text in the `textarea` element, press the *Submit* button, and our comment gets appended to the list. However, notice what happens when we enter HTML in the field, like `React is <b>awesome</b>!`:

{% include image.html name="html_bold.png" alt="Entering text with an HTML bold tag in our form field" %}
{% include image.html name="list_with_html_bold.png" alt="Rendering the list of comments and showing the HTML bold tag" %}

That's a cool feature, right? The user can style their comments however they want!

Well, let's try entering another bit of HTML, `<img src="nope.jpg" onerror="alert('Hacked!')" />`:

{% include image.html name="html_img.png" alt="Enter html image tag in form field" %}
{% include image.html name="alert.png" alt="Inline scripting code is executed" %}

Oops, we've been hacked! How did that happen? If we look at the HTML we entered, we'll see that the `<img>` tag pointed to a non-existent image, so the `onerror` handler ran and executed the JavaScript code we had inlined there. If our app were storing that comment in a database, and displaying it every time a user visited the page, we'd have a big problem in our hands!

*(Try playing with the form yourself, see if you can find other ways to execute code through that form. If you want spoilers, look at the [OWASP XSS cheat sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet).)*

React tries to protect us from XSS attacks by escaping all strings we render as children of an element. If we really want to render unescaped HTML, we need to use a special prop appropriately named [`dangerouslySetInnerHTML`](https://reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml):

```js
export default class ListOfComments extends React.Component {
  render() {
    const { comments } = this.props;

    return (
      <ul>
        {comments.map((comment) => (
          <li
            key={comment}
            dangerouslySetInnerHTML={ { __html: comment } }
          />
        ))}
      </ul>
    );
  }
}
```

While there are valid use cases for `dangerouslySetInnerHTML`, such as rendering sanitized HTML coming from your server, I'd try to minimize its use, because it's really easy to do something wrong and open ourselves up for an attack.

## Content Security Policy

The problem underlying an XSS attack is that the browser doesn't know which sources of code to trust. My `<script src="https://cdn.jsdelivr.net/npm/react@latest/umd/react.production.min.js"></script>` tag and your injected `<img src="nope.jpg" onerror="alert('Hacked!')" />` tag are both valid, so the browser will execute them all.

Content Security Policy is a security feature [present in all modern browsers](https://caniuse.com/#search=CSP) that allows us to list which sources are trusted by our web application, so that the browser is able to block everything else. If your name is on the guest list, you can get into the party; otherwise, you're staying outside.

There's two ways to declare a Content Security Policy: through a `Content-Security-Policy` HTTP header when serving your HTML page, and through a `<meta http-equiv="Content-Security-Policy">` tag.

In our sample app we don't have control over HTTP headers, so we'll use a `<meta>` tag. Let's add the most restrictive Content Security Policy possible, `default-src 'none'`, which basically tells the browser to not trust any source, and see what happens:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Security-Policy"
          content="default-src 'none'" />
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

Well, it looks like everything broke:

{% include image.html name="csp_errors.png" alt="Content Security Policy errors" %}

The error messages are somewhat cryptic, but we can figure them out by looking things up in the [Content Security Policy article in MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy), which describes all possible directives and values we can specify.

Let's go one by one.

### Fixing script errors

The first error has to do with scripts getting blocked:

> Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "default-src 'none'". Note that 'script-src' was not explicitly set, so 'default-src' is used as a fallback.

We didn't provide a `script-src` policy, so the browser fell back to what's specified as `default-src` (which is `'none'`) and refused to load all scripts. It looks like the folks hosting our sample app, [StackBlitz](https://stackblitz.com/), are running our code through `eval`, so we'll need to allow that as a source in our `script-src` directive:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; script-src 'unsafe-eval'" />
```

(Note that you wouldn't want to do this in a real app. Allowing any of the `unsafe-*` sources like `unsafe-eval` or `unsafe-inline` is a probably a really bad idea!)

### Fixing style errors

The second error has to do with styles getting blocked:

> Refused to load the stylesheet 'https://cdn.jsdelivr.net/npm/@progress/kendo-theme-default@latest/dist/all.css' because it violates the following Content Security Policy directive: "default-src 'none'". Note that 'style-src' was not explicitly set, so 'default-src' is used as a fallback.

Same thing, we didn't provide a `style-src` policy, so the browser fell back to `default-src` and refused to load all styles. We can specify the exact path to the stylesheet, a partial path, or just the domain. We'll specify a partial path here, so that all styles coming from `@progress`-scoped packages are trusted:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; script-src 'unsafe-eval'; style-src https://cdn.jsdelivr.net/npm/@progress/" />
```

### Fixing font errors

The final errors have to do with fonts getting blocked:

> Refused to load the font 'data:font/ttf;base64,...' because it violates the following Content Security Policy directive: "default-src 'none'". Note that 'font-src' was not explicitly set, so 'default-src' is used as a fallback.

The CSS file above seems to be inlining base64-encoded fonts using the `data:` protocol, so we'll need to add a `font-src` directive that allows for that:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; font-src data:; script-src 'unsafe-eval'; style-src https://cdn.jsdelivr.net/npm/@progress/" />
```

### Trial by fire

Now the app is working again! Let's try our original attack by injecting `<img src="nope.jpg" onerror="alert('Hacked!')" />` into the page:

{% include image.html name="alert_blocked.png" alt="Content Security Policy blocked our XSS attack!" %}

CSP blocked the attack! Here's the error message we got:

> Refused to execute inline event handler because it violates the following Content Security Policy directive: "script-src 'unsafe-eval'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution.

We didn't allow inline scripts as a trusted source, so the browser prevented the code from running. 🎉🎉🎉

## Report-Only Mode

Ok, this is really cool, but there's no way I'm going to add a Content Security Policy to my app! What if I misconfigure it and I break some important feature for my users?

Well, the people behind the CSP spec must have thought of that exact scenario, and added a report-only mode that will output errors to the console, but not block execution. So if we were to add this tag to our app:

```html
<meta http-equiv="Content-Security-Policy-Report-Only"
      content="default-src 'none'" />
```

We would see all the CSP violations happening in the app, but it would keep working just fine.

We can go one step further, and use the `report-uri` directive to tell the browser to send all violations to an endpoint under our control, so that we can aggregate them in a dashboard or something:

```html
<meta http-equiv="Content-Security-Policy-Report-Only"
      content="default-src 'none'; report-uri /api/csp " />
```

Whenever the browser encounters a violation of our CSP, it will make a `POST` request to `/api/csp` with content that looks like this:

```json
{
  "csp-report": {
    "document-uri": "https://www.example.com",
    "referrer": "",
    "violated-directive": "img-src",
    "effective-directive": "img-src",
    "original-policy": "default-src 'none'; report-uri /api/csp",
    "disposition": "report",
    "blocked-uri": "https://www.evil.com",
    "status-code": 0,
    "script-sample": ""
  }
}
```

## Conclusion

Content Security Policy is a really powerful tool that modern browsers provide to prevent Cross-Site Scripting attacks. My advice is to start with the most restrictive policy possible, `default-src 'none'`, and go from there. Use `Content-Security-Policy-Report-Only` to aggregate reports and ensure your CSP is configured correctly, so that you don't break any functionality for your users. Once you're sure everything is configured correctly, switch to `Content-Security-Policy` to enforce the policy for real, and stop the bad guys from doing their evil.
