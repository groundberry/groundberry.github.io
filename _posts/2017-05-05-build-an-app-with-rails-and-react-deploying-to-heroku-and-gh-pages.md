---
layout: post
title:  "Build an app with Rails and React - Deploying to Heroku and GitHub Pages"
date:   2017-05-05
categories: development
---

This is the fourth post of a series of four where I'll show the main steps I followed to build [Flashcards](https://groundberry.github.io/flashcards-client/), a single page app that will help users study and learn any subject they want through spaced repetition.

- Part 1 - [The back-end]({% post_url 2017-03-18-build-an-app-with-rails-and-react-the-back-end %})
- Part 2 - [User authentication]({% post_url 2017-04-08-build-an-app-with-rails-and-react-user-authentication %})
- Part 3 - [The front-end]({% post_url 2017-04-23-build-an-app-with-rails-and-react-the-front-end %})
- Part 4 - Deploying to Heroku and GitHub Pages

The back-end is a [Rails](http://rubyonrails.org/) app that will store the information provided by the users. We'll deploy it to [Heroku](https://www.heroku.com/).
The front-end is built with [React](https://facebook.github.io/react/), a JavaScript library for user interfaces. We'll deploy it to [GitHub Pages](https://pages.github.com/).

## Deploying to Heroku

Let's install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) with Homebrew to deploy from the command line:

```
$ brew install heroku
```

We'll log in using our credentials:

```
$ heroku login
Enter your Heroku credentials.
Email: youremailhere@example.com
Password (typing will be hidden):
Authentication successful.
```

Since our app is built with Rails it comes with a `config.ru` file with the following content, so we don't have to create it:

```ruby
# config.ru

require_relative 'config/environment'

run Rails.application
```

From our app directory we'll create a Heroku app in the cloud:

```
$ heroku create flashcards-server
Creating app... done, ⬢ flashcards-server
https://flashcards-server.herokuapp.com/ | https://git.heroku.com/flashcards-server.git
```

If we don't specify a name for our app, Heroku will assign a random one that can be changed later.

We can check our remotes by typing `git remote -v`:

```
$ git remote -v
heroku  https://git.heroku.com/flashcards-server.git (fetch)
heroku  https://git.heroku.com/flashcards-server.git (push)
```

Now let's push our changes to that remote so that they are deployed:

```
$ git push heroku master
```

Remember to run migrations every time we deploy new changes:

```
$ heroku run rake db:migrate && heroku restart
```

If you remember, when we were dealing with [user authentication]({% post_url 2017-04-08-build-an-app-with-rails-and-react-user-authentication %}), we were working with environment variables. Now we need to [set them up in Heroku](https://devcenter.heroku.com/articles/config-vars) for our deployed app. We can do it from the Heroku dashboard, or from our command line (notice that we don't enter spaces around the = sign):

```
$ heroku config:set CLIENT_ID="XXXXXXXXXXXXXXXXXXXX"
$ heroku config:set CLIENT_SECRET="YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
$ heroku config:set JWT_SECRET="ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ"

$ heroku config:set GITHUB_ACCESS_TOKEN_URL="https://github.com/login/oauth/access_token"
$ heroku config:set GITHUB_USER_INFO_URL="https://api.github.com/user"

$ heroku config:set FLASHCARDS_CLIENT_URL="https://groundberry.github.io/flashcards-client/"
```

We can check the value for all our config variables by typing:

```
$ heroku config
```

Our server is live and up in the cloud! ☁️

## Deploying to GitHub Pages

Since we have separated our app into front-end and back-end, we'll deploy them to different hosts. Our client will be hosted on GitHub Pages.

If we run the command `npm run build` in our project directory, it'll suggest we specify the `homepage` of our app in our `package.json`.

If we run `npm run build` again, we'll be asked to install `gh-pages`:

```
$ npm install --save-dev gh-pages
```

Finally, we'll add a `deploy` task to our `package.json` that builds all assets (JavaScript and CSS bundles), and deploys them.

```json
{
  "name": "flashcards-client",
  "version": "0.1.0",
  "private": true,
  "homepage": "https://yourusername.github.io/flashcards-client",
  "devDependencies": {
    "gh-pages": "^0.12.0",
    "react-scripts": "0.8.4"
  },
  "dependencies": {
    "react": "^15.4.1",
    "react-dom": "^15.4.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "deploy": "npm run build && gh-pages -d build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

Now we can deploy our app by running:

```
$ npm run deploy
```

If we access <https://yourusername.github.io/flashcards-client> we'll see the app. Great success! 🙆
