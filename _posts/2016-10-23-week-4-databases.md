---
layout: post
title:  "Week 4 - Databases"
date:   2016-10-23
categories: makers-academy course
---

## The week

In week four we learnt how our browser requests information from our server written using [Sinatra](http://www.sinatrarb.com/) and how the server takes information from a database in order to respond to this `GET` request. We learnt how to work with [PostgreSQL](https://www.postgresql.org/), an open source object-relational database system. During the weekend challenge we implemented a signup flow, by saving the user name, email and hashed password into a database. We used [bcrypt](https://github.com/codahale/bcrypt-ruby) to hash the user's password.

## What I learnt

I learned a lot about [DataMapper](http://datamapper.org/), an object-relational mapper written in Ruby, and [Database Cleaner](https://github.com/DatabaseCleaner/database_cleaner), a set of strategies for cleaning our database during tests. It was a bit hard finding the right information from these resources though. When we finished our app we pushed it to [Heroku](https://www.heroku.com/), a service that enables developers to deploy an app to the cloud just by doing a `git push`.

## Struggles

Databases week has been the hardest since we started the course back in September. I struggled a lot trying to grasp where the information that we store in the databases is saved in the computer and how to access it. After a lot of diagramming and reading I have a slightly better understanding of it, but I still need to dig deeper. Now I know a bit more how all the different actors interact with each other but practice makes perfect.
The weekend challenge was also quite hard, but exciting. Since I didn't have time during the week to cover users signing up and signing in, I was pretty scared of having to deal with this tricky part of web app development on my own... I started the challenge facing some setup difficulties but I ended up managing to create a basic version of the app.

I'm looking forward to starting week 5 and learning JavaScript, the front-end language that can also be used on the server side. Exciting!
