---
layout: post
title:  "Adding Google Analytics to Jekyll"
date:   2016-09-16
categories: setup
---

I wanted to monitor the traffic on my blog. You know, where visits come from, how much time they spend on each article, etc. Google has a product called [Google Analytics](https://analytics.google.com/) that allows us to do just that, by including a bit of JavaScript code in our page. Because I'm using the [Minima theme with Jekyll](https://github.com/jekyll/minima), I didn't even need to deal with that!

## Creating a Google Analytics account

In order to start using Google Analytics, we'll need to create an account. If we hit <https://analytics.google.com/> and log in with our Google credentials, we'll see a page like this one:

{% include image.html name="sign_up.png" alt="Sign up page in Google Analytics" %}

We'll click the *Sign Up* button, and enter the information for the website we want to track:

{% include image.html name="create_account.png" alt="Creating a new account" %}

Once we are done with that, we'll be presented with our tracking ID, which we'll need afterwards:

{% include image.html name="tracking_id.png" alt="Getting our tracking ID" %}

## Adding Google Analytics to our site

The Minima theme I'm using with Jekyll allows you to [enable Google Analytics really easily](https://github.com/jekyll/minima#enabling-google-analytics). We just need to add a `google_analytics` parameter to our `_config.yml` file. Remember that tracking ID we got before? Now's the time to use it:

```
title: My Blog
...
google_analytics: UA-12345678-1
```

After publishing our changes, we'll start receiving data for our site!

{% include image.html name="receiving_data.png" alt="Receiving data" %}
