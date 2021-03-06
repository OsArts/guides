---
title: "Overview"
order: 10
---

Hanami is based on two principles: [Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) and [Monolith First](http://martinfowler.com/bliki/MonolithFirst.html).

## Clean Architecture

The main purpose of this architecture is to enforce a **separation of concerns** between the **core** of our product and the **delivery mechanisms**.
The first is expressed by the set of **use cases** that our product implements, while the latter are interfaces to make these features available to the outside world.

When we generate a new project we can find two important directories: `lib/` and `apps/`.
They are home to the main parts described above.

### Application Core

We implement a set of functionalities, without worrying about how they can be exposed to the outside world.
This is the **cornerstone** of our product, and we want to be careful on how we manage dependencies for it.

`Hanami::Model` is the default choice for persisting our Ruby objects.
This is a _soft-dependency_, it can be removed from our `Gemfile` and replaced with something else.

Let's have a look at how the `lib/` directory appears for a new generated project called `bookshelf` that uses `Hanami::Model`.

```shell
$ tree lib
lib
├── bookshelf
│   ├── entities
│   ├── mailers
│   │   └── templates
│   └── repositories
└── bookshelf.rb

5 directories, 1 file
```

The idea is to develop our application like a Ruby gem.

The `lib/bookshelf.rb` file is the entry point for our application, when we require this file, we require and initialize all the code under `lib/`.

There are two important directories:

  * `lib/bookshelf/entities`
  * `lib/bookshelf/repositories`

They contain [entities](/guides/1.2/entities/overview) that are Ruby objects at the core of our model domain, and they aren't aware of any persistence mechanism.
For this purpose we have a separate concept, [repositories](/guides/1.2/repositories/overview), which are a mediator between our entities and the underlying database.

For each entity named `Book` we can have a `BookRepository`.

We can add as many directories that we want, such as `lib/bookshelf/interactors` to implement our use cases.

### Delivery Mechanisms

Hanami generates a default application named `Web`, which lives under `apps/web`.
This application **depends** on the core of our product, as it uses entities, repositories and all the other objects defined there.

It's used as web delivery mechanism, for our features.

```shell
$ tree apps/web
apps/web
├── application.rb
├── assets
│   ├── favicon.ico
│   ├── images
│   ├── javascripts
│   └── stylesheets
├── config
│   └── routes.rb
├── controllers
├── templates
│   └── application.html.erb
└── views
    └── application_layout.rb

8 directories, 5 files
```

Let's have a quick look at this code.

The file `apps/web/application.rb` contains a Hanami application named `Web::Application`, here we can configure all the settings for this **component** of our project.
Directories such as `apps/web/controllers`, `views` and `templates` will contain our [actions](/guides/1.2/actions/overview), [views](/guides/1.2/views/overview) and [templates](/guides/1.2/views/templates).

Web assets such as javascripts and stylesheets will be automatically served by the application.

## Monolith First

Our default application `Web` can be used as a UI interface for our customers.
At a certain point in our story, we want to manage our users with an admin panel.

We know that the set of features that we're going to introduce doesn't belong to our main UI (`Web`).
On the other hand, it's **too early** for us to implement a microservices architecture, only for the purpose of helping our users reset their password.

Hanami has a solution for our problem: we can generate a new app that lives in the same Ruby process, but it's a separated component.

```shell
$ bundle exec hanami generate app admin
```

This command MUST be run from the root of our project. It will generate a new application (`Admin::Application`) under `apps/admin`.

In the late stages of our product life-cycle, we could decide to extract this into a standalone component.
We would just need to move everything under `apps/admin` into another repository and deploy it separately.

## Anatomy Of A Project

We have already examined `lib/` and `apps/`, but there are other parts of a newly generated project that deserve to be explained.

```shell
$ tree -L 1
.
├── Gemfile
├── Gemfile.lock
├── README.md
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
└── spec
```

Let's quickly introduce them:

  * `Gemfile` and `Gemfile.lock` are [Bundler](http://bundler.io) artifacts
  * `README.md` tells us how to setup and use the project.
  * `Rakefile` describes Rake task for our project.
  * `config/` contains an important file `config/environment.rb`, which is the **entry point** for our project.
    By requiring it, we'll preload our dependencies (Ruby gems), Hanami frameworks and our code.
  * `config.ru` is a file that describes how a Rack server must run our applications.
  * `db/` contains database files (for File System adapter or SQLite).
    When our project uses a SQL database it also contains `db/migrations` and `db/schema.sql`.
  * `spec/` contains unit and acceptance tests.
