---
layout: post
title:  "Build an app with Rails and React - The back-end"
date:   2017-03-18
categories: development
---

This is the first post of a series of four where I'll show the main steps I followed to build [Flashcards](https://groundberry.github.io/flashcards-client/), a single page app that will help users study and learn any subject they want through spaced repetition.

- Part 1 - The back-end
- Part 2 - [User authentication]({% post_url 2017-04-08-build-an-app-with-rails-and-react-user-authentication %})
- Part 3 - The front-end
- Part 4 - Deploying to Heroku and GitHub pages

The back-end is a [Rails](http://rubyonrails.org/) app that will store the information provided by the users. The front-end is built with [React](https://facebook.github.io/react/), a JavaScript library for user interfaces.

## Building our back-end on Rails

The first thing we have to do to build our Rails app is install both Ruby and Rails. You can find out how to install Ruby in my first post [Setting up my development environment]({% post_url 2016-08-29-setting-up-my-development-environment %}). Now let's install Rails.

```
$ gem install rails
```

Instead of using Rails to generate the HTML pages, our back-end will just generate JSON which will be consumed by our front-end through Ajax requests. So when we call `rails new` we'll pass this flag `--api` that removes a ton of stuff that we won't use.

```
$ rails new flashcards --api
```

Rails comes with `sqlite3` as the default database for Active Record. However in production we want to use [PostgreSQL](https://www.postgresql.org/). Let's add it to our `Gemfile`:

```ruby
source 'https://rubygems.org'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 5.0.0', '>= 5.0.0.1'
# Use postgresql as the database for Active Record
gem 'pg', '~> 0.18'
# Use sqlite3 as the database for Active Record
# gem 'sqlite3'
```

Now we install this new dependency with `bundler`:

```
$ bundle install
```

We also have to change the default database settings in our `config/database.yml` file to point to our databases:

```yaml
default: &default
  adapter: postgresql
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: flashcards_development

test:
  <<: *default
  database: flashcards_test

production:
  <<: *default
  database: flashcards_production
```

Rails already knows the name of the database from our `config/database.yml`, so running `rake db:create` will create it if it doesn't exist. We also need to run all our migrations with `rake db:migrate`. If we combine the two:

```
$ rake db:create db:migrate
```

We'll need to do the same for our test database. The way we tell `rake` to run these tasks against it is by setting the `RAILS_ENV` environment variable to `test`:

```
$ RAILS_ENV=test rake db:create db:migrate
```

## Creating the models of our app

The models are Ruby classes that hold the business logic for our app. We'll build four models containing the methods that will allow the user to interact with their flashcards.

### Users

Let's create a `user` model with `login` and `name` fields, and migrate our database:

```
$ rails generate scaffold user login:string name:string
$ rake db:migrate
```

By having generated the `user` with `scaffold` instead of `model`, we not only get a model, but also a route and a controller. Nice!

[Controllers](http://guides.rubyonrails.org/action_controller_overview.html) are responsible for producing the response that matches the request that the user made.

[Routes](http://guides.rubyonrails.org/routing.html) map a URL requested by a user to an action in a controller. Go ahead and check what routes are active now by typing `rake routes` in the console:

```
$ rake routes
Prefix Verb   URI Pattern          Controller#Action
 users GET    /users(.:format)     users#index
       POST   /users(.:format)     users#create
  user GET    /users/:id(.:format) users#show
       PATCH  /users/:id(.:format) users#update
       PUT    /users/:id(.:format) users#update
       DELETE /users/:id(.:format) users#destroy
```

Here's the meaning of each column:

- Prefix: Prefix used in route helpers (e.g. `users_url`).
- Verb: Type of request that is made to the server.
- URI Pattern: Pattern used to match URLS.
- Controller#Action: Controller and action that will get invoked when this URL is hit.

If you look at the output of the `rails generate scaffold` command, you'll also notice we got tests for free. The following files were created:

- `test/fixtures/users.yml`: User fixtures. We'll use fixtures to load fake data in our tests. You can edit this data however you want.
- `test/models/user_test.rb`: Unit tests for the user model.
- `test/controllers/users_controller_test.rb`: Integration tests for the user controller.

If we run `rake`, all these tests will get executed:

```
$ rake
Run options: --seed 41275

# Running:

.....

Finished in 0.629063s, 7.9483 runs/s, 11.1277 assertions/s.

5 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

We can also play with our first model manually, and create new users from the console:

```
$ rails console
irb(main):001:0> user = User.new(login: 'groundberry', name: 'Blanca')
irb(main):002:0> user.save!
```

If we want to play with the `/users` endpoint, we first need to start the server:

```
$ rails server
```

Now we can check the endpoint using `curl` (or entering the URL in our browser):

```
$ curl http://localhost:3000/users
[{"id":1,"login":"groundberry","name":"Blanca","created_at":"2017-03-19T01:51:16.813Z","updated_at":"2017-03-19T01:51:16.813Z"}]
```

According to the information we saw in `rake routes`, the `/users` request got forwarded to the `index` action of the `UsersController` controller, which rendered the response as JSON. In this case, we got back an array with the information for all the users stored in our database.

In a similar way, we can delete entries from the console just searching the user by its `id`:

```
irb(main):003:0> user = User.find_by_id(1)
irb(main):004:0> user.destroy!
```

It seems that everything is working as expected. Awesome!

### Flashcards

Next we'll create the `flashcard` model. Each flashcard will belong to the user that created it, and will have a question on one side and the correct answer on the other side. We'll define both `question` and `answer` as `text` instead of `string`, so that we can enter longer pieces of information:

```
$ rails generate scaffold flashcards user:references question:text answer:text
```

If we look at the generated `flashcard` model, we'll see that a `belongs_to` association has been added from flashcard to user:

```ruby
# app/models/flashcard.rb

class Flashcard < ApplicationRecord
  belongs_to :user
end
```

What Rails doesn't create automatically is the association from user to flashcards, so we need to add this manually. One user can create many flashcards, so we'll add the association `has_many :flashcards` to the `user` model:

```ruby
# app/models/user.rb

class User < ApplicationRecord
  has_many :flashcards
end
```

Now we run our migrations once more to update the schema in our database:

```
$ rake db:migrate
```

We currently have two migrations, and we can check the resulting schema by looking at the file `db/schema.rb`. It's important that we don't manually modify neither the schema file nor existing migrations. If we want to add a new column to a table, or remove an existing one, we should create a new migration and update our schema.

We can run our tests now with the command:

```
$ rake
```

We'll see an error! ❌

```
Error:
UsersControllerTest#test_should_destroy_user:
ActiveRecord::InvalidForeignKey: PG::ForeignKeyViolation: ERROR:  update or delete on table "users" violates foreign key constraint "fk_rails_47a5d025da" on table "flashcards"
```

What this error message is telling us is that, by trying to delete one of the sides of the association, we're breaking the database constraints. We have to make sure that, when we delete a user, all the flashcards linked to this user are being deleted as well. To do so we use the `dependent` option of the association:

```ruby
# app/models/user.rb

class User < ApplicationRecord
  has_many :flashcards, dependent: :destroy
end
```

We run our tests again and... All test are passing! ✅

### Tags

We want to be able to select only the flashcards that relate to a specific subject, so we should be able to tag them. To do that we'll need to create a `tag` model:

```
$ rails g scaffold tag user:references name:string
```

```ruby
# app/models/tag.rb

class Tag < ApplicationRecord
  belongs_to :user
end
```

Again, Rails doesn't create automatically the association from user to tags, so we need to add this manually. One user can create many tags, so we'll add the association `has_many :tags` to the `user` model:

```ruby
# app/models/user.rb

class User < ApplicationRecord
  has_many :flashcards, dependent: :destroy
  has_many :tags
end
```

We have created a new model so we need to run our migrations again:

```
$ rake db:migrate
```

If we run `rake`, we'll encounter an error similar to the one we got when we created the `flashcard` model, so we can easily fix that.

```
Error:
UsersControllerTest#test_should_destroy_user:
ActiveRecord::InvalidForeignKey: PG::ForeignKeyViolation: ERROR:  update or delete on table "users" violates foreign key constraint "fk_rails_e689f6d0cc" on table "tags"
```

You know what to do here.

```ruby
# app/models/user.rb

class User < ApplicationRecord
  has_many :flashcards, dependent: :destroy
  has_many :tags, dependent: :destroy
end
```

Now all our test are passing again!

### Taggings

One flashcard can have many tags, and one tag can have many flashcards. This is what's known as a many-to-many association between flashcards and tags. There is no way to represent a many-to-many association in the database, so what we can do is split it into two one-to-many associations. To do that we'll create an intermediate model called `tagging` that connects a flashcard and a tag:

```
$ rails g model tagging flashcard:references tag:references
```

We've used `model` instead of `scaffold` because we don't need a controller and routes for it.

A tagging belongs to a flashcard and a tag at the same time:

```ruby
# app/models/tagging.rb

class Tagging < ApplicationRecord
  belongs_to :flashcard
  belongs_to :tag
end
```

On the other hand, a flashcard can have many taggings:

```ruby
# app/models/flashcard.rb

class Flashcard < ApplicationRecord
  belongs_to :user

  has_many :taggings, dependent: :destroy
  has_many :tags, through: :taggings
end
```

And a tag can also have many taggings:

```ruby
# app/models/tag.rb

class Tag < ApplicationRecord
  belongs_to :user

  has_many :taggings, dependent: :destroy
  has_many :flashcards, through: :taggings
end
```

Now all our models are linked through associations and all our tests are passing.

## Validations

We'll add [validations](http://guides.rubyonrails.org/active_record_validations.html) to our `flashcard`, `tag`, and `user` models, so that they don't get into weird states.

We want to make sure that any time we create a new flashcard it'll have a question and an answer, so we'll add the following to the `flashcard` model:

```ruby
# app/models/flashcard.rb

class Flashcard < ApplicationRecord
  # ...

  validates :question, presence: true
  validates :answer, presence: true
end
```

In a similar way, we want to make sure that any time we add a new tag to a flashcard, it has a name, and this tag name is unique to he user that created it, so we don't have two repeated tags for a particular user.

```ruby
# app/models/tag.rb

class Tag < ApplicationRecord
  # ...

  validates :name, presence: true, uniqueness: { scope: :user_id }
end
```

Lastly, we'll add validations to our `user` model so that every new user that registers in our app will have a name, and a unique login.

```ruby
# app/models/user.rb

class User < ApplicationRecord
  has_many :flashcards, dependent: :destroy
  has_many :tags, dependent: :destroy

  validates :name, presence: true
  validates :login, presence: true, uniqueness: true
end
```

If we run our tests now we'll get three errors. Let's fix them.

We need to change the `test/users_controller_test`, so that the uniqueness validations don't break everything:

```ruby
# test/users_controler_test.rb

class UsersControllerTest < ActionDispatch::IntegrationTest
  # ...

  test "should create user" do
    assert_difference('User.count') do
      post users_url, params: { user: {
        login: 'someone',
        name: 'someone'
      } }, as: :json
    end

    assert_response 201
  end

  test "should update user" do
    patch user_url(@user), params: { user: {
      login: 'someone',
      name: 'someone'
    } }, as: :json
    assert_response 200
  end

  #...
end
```

We have to do something similar in our `tags_controller_test` since we are creating a new tag each time.

```ruby
# test/tags_controler_test.rb

class TagsControllerTest < ActionDispatch::IntegrationTest
  # ...

  test "should create tag" do
    assert_difference('Tag.count') do
      post tags_url, params: {
        tag: {
          name: 'Some Tag',
          user_id: @tag.user_id
        }
      }, as: :json
    end

    assert_response 201
  end

  test 'should update tag' do
    patch tag_url(@tag), params: {
      token: @token,
      tag: {
        name: 'Some Tag',
        user_id: @tag.user_id
      }
    }, as: :json
    assert_response 200
  end

  # ...
end
```

Now all our tests are green again!

## Nested routes

One last thing... We need to update our `flashcards` and `tags` controllers so that we can render the tags for a flashcard, and the flashcards associated to a tag.

Let's start by adding a method to get the tags for a flashcard:

```ruby
# app/controllers/flashcards_controller.rb

class FlashcardsController < ApplicationController
  before_action :set_flashcard, only: [:show, :tags, :update, :destroy]

  # ...

  # GET /flashcards/1/tags
  def tags
    @flashcard.tags
  end

  # ...
end
```

Now we need to map a route to that action. To do so, we'll edit our `config/routes.rb` file, and nest a new route inside the `flashcards` resource:

```ruby
# config/routes.rb

Rails.application.routes.draw do
  # ...

  resources :flashcards do
    get :tags, on: :member
  end
end
```

If we now run `rake routes`, we'll see the new route pointing to `flashcards#tags`:

```
$ rake routes
        Prefix Verb   URI Pattern                    Controller#Action
...
tags_flashcard GET    /flashcards/:id/tags(.:format) flashcards#tags
```

Now we'll do the same thing to get the flashcards associated to a tag:

```ruby
# app/controllers/tags_controller.rb

class TagsController < ApplicationController
  before_action :set_tag, only: [:show, :flashcards, :update, :destroy]

  # ...

  # GET /tags/1/flashcards
  def flashcards
    @tag.flashcards
  end

  # ...
end
```

And we'll edit our `config/routes.rb` in a similar way:

```ruby
# config/routes.rb

Rails.application.routes.draw do
  # ...

  resources :tags do
    get :flashcards, on: :member
  end
end
```

Running `rake routes` will show the new route pointing to `tags#flashcards`:

```
$ rake routes
        Prefix Verb   URI Pattern                    Controller#Action
...
flashcards_tag GET    /tags/:id/flashcards(.:format) tags#flashcards
```

Now we can `curl` for both:

```
$ curl http://localhost:3000/flashcards/2/tags
$ curl http://localhost:3000/tags/7/flashcards
```

## Improving our app

Everything seems to work fine, right? Well, if you have a look at both `flashcards_controller` and `tags_controller`, they are currently showing all the existing flashcards and tags, not just the ones that belong to the user that created them. How can we fix that?

We'll cover this in another post. We need to authenticate the user first! 🔑 🔓
