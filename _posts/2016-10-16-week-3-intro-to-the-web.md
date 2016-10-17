---
layout: post
title:  "Week 3 - Intro to the web"
date:   2016-10-16
categories: makers-academy course
---

## The week

This week we discovered how a Ruby app is seen on the web. We learnt about the relationship between a client and a server, and how a request made by the former is sent to the latter using HTTP. The server processes this request and sends the required information back to the client (the browser) as HTML that will be displayed to the user through a visual interface. Awesome!

## What I learnt

We started the week by reading documentation about all these topics, and a group of colleagues performed the request-response dance... literally! It helped me understand and visualise the flow of information between server and client, and remember the format in which the information is transferred on the web.

To put this knowledge into practice, we worked on a web app during the week, where two players attacked each other reducing their points, until one of them got to zero. Then a message was shown, telling who won and who lost.

We built it using [Sinatra](http://www.sinatrarb.com/), a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) for quickly creating web applications in Ruby. We first implemented the domain logic, splitting it in different classes following the single-responsibility principle, in order to keep each one focussing on only one thing. We then created the controllers for the different routes in the web app, and their corresponding views. We tried to keep the route controllers as thin as possible, pushing the business logic to the domain models.

Everything was done following [TDD](https://en.wikipedia.org/wiki/Test-driven_development). The domain logic was tested using [RSpec](http://rspec.info/). For the views we used [Capybara](http://jnicklas.github.io/capybara/) to simulate how the user interacts with our application.

When the logic was finished and the app worked, we implemented the front-end of our app using [HTML](https://developer.mozilla.org/es/docs/Web/HTML) and [CSS](https://developer.mozilla.org/es/docs/Web/CSS). This is a very creative part of the web app development process, where you can play around with font families, colours, layouts, and so on. The sky is the limit!

## Struggles

This week I really struggled when working on the weekend challenge, but the best thing that I took out of it was knowing where I need to focus, in order to build on top of solid foundations. I wrote down a list of things to review, such as doubles and mocks, variable scope, debugging, etc. During the week I managed to find time and go through each one of them in the evenings. It helped me feel more confident, but I still need to keep working on them.

It was a very exciting week. It's great to see how your hard work looks like when you open it in a web browser!
