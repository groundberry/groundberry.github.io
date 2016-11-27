---
layout: post
title:  "Week 8 - Introduction to Rails"
date:   2016-11-20
categories: makers-academy course
---

## The week

This week we used [Rails](http://rubyonrails.org/), a web application framework, to build a clone of Yelp, an app for reviewing restaurants. Rails prioritises convention over configuration. This means that Rails structures our code in the way it thinks is best, assuming certain aspects of our web app's design and architecture. This structure saves us lots of time and allows us to have a decent app up and running quite quickly.

On the flip side, due to amount of things Rails is doing for us, our app can be hard to debug, since many methods and processes occur behind the scenes.

We used [Devise](https://github.com/plataformatec/devise), to help us manage users, and [Omniauth](https://github.com/omniauth/omniauth), to allow users to log into our app using external application credentials such as Facebook, Twitter or Google.

## What I learnt

It was my first contact with Rails, and it was very exciting to build our first app using a framework that provides a clear structure but at the same time a lot of magic. I learnt how a Rails app works, how all the files and folders interact with each other, and the most important thing when you work with Rails, to embrace the magic. 🚂

## Struggles

Managing users, and dealing with sign up/in/out, using Devise and Omniauth was really fun, but we also encountered a few challenges.

The magic behind Rails helps you set up an app quickly, but you also have to be aware of the methods that are already in place so that you don't overwrite them. This was one of my greatest struggles. I spent hours trying to figure out why I could sign into my app using Facebook login, but then I couldn't leave! What happened was that I created a method with the same name as an existing Devise helper, which broke the sign out functionality, and prevented me from destroying the session.

Once I found out which method was causing the issue, the logout button suddenly appeared, and all my troubles were gone! I had a pretty nice app that allowed users to sign up, view restaurants, leave a review, and leave. Awesome!
