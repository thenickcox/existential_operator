---
title: Testing Ember Models With Jasmine, CoffeeScript, and Rails 4
date: 2014-01-04 18:58 UTC
layout: post
---

## TL;DR

Getting Ember set up with Jasmine and Rails 4 is tricky. I've done the hard part and set up a skeleton Rails app that contains a passing Ember model test as a starting point for your apps. Feel free to [fork the repo](https://github.com/thenickcox/ember_jasmine_setup) and start a project if you want to use Jasmine to test your Ember app. For more detail about how to get it working, and a pretty thorough run through of some high- and low-level Ember concepts, read on.

[READMORE]

It's no secret that, while test-driven development is in theory very simple and straightforward, getting it set up properly is anything but. As I've said in the past, [TDD is 90% getting your test framework set up](https://twitter.com/everydaytype/status/397423602070327297).

I've been following along with [Noel Rappin](http://www.noelrappin.com/)'s great book, Mastering Space and Time With JavaScript. It's one of the better books I've read on Ember (I've [read](http://www.manning.com/skeie/) [two](http://gilesbowkett.blogspot.com/2013/06/heretical-guide-to-ember-js.html) that I found disappointing, owing both to the difficulty of following them, and of Ember's quickly changing API prior to its 1.0 release). And while he's updating it relatively frequently, I got stuck on a piece of outdated code based on a pre-1.0 version of Ember Data.

I thought to myself, _If I get this figured out, I'm going to document it so that no one has to deal with this same frustration._ Well, with the help of a few Stack Overflow [posts](http://stackoverflow.com/questions/19578226/testing-ember-data-1-0-0-with-jasminejs), I got the very first Ember model test passing. And, as we all know, the first test is always the hardest to get passing. If you can get there, it's smooth sailing. Or, at least, it's easier.

Given that we've finally arrived at Ember 1.0 and the [API is stable](http://emberjs.com/blog/2013/08/31/ember-1-0-released.html), it's my hope that this post will contain working code for at least a few months, and, if followed step-by-step, will produce working results.

A few notes before we begin, we'll be using Rails, CoffeeScript (specs read better without the noise of semicolons and curly brackets), and the `ember-rails` gem. Any amount of this code may be helpful outside of this context, but this code is very targeted to get a working test setup using these exact tools.

Additionally, while this code doesn't assume any Ember knowledge (it just involves getting set up to test it), it helps to have a working familiarity with Rails (duh), the command line, and CoffeeScript syntax.

## Getting Started

### Gems

Let's get started by starting a new Rails application. Open up the command line and run `rails new jasmine_setup`. Next, we'll need to get our gemfile set up properly. Here's a look at all the gems we'll be including, with an explanation below:

<pre>
  <code>
    source 'https://rubygems.org'

    gem 'rails', '4.0.1'
    gem 'sqlite3'

    gem 'sass-rails', '~> 4.0.0'
    gem 'coffee-rails', '~> 4.0.0'

    gem 'uglifier', '>= 1.3.0'

    gem 'jquery-rails'
    gem 'ember-rails'
    gem 'ember-source', '1.2.0'
    gem 'ember-data-source', '1.0.0.beta.4'

    gem 'jasmine'
    gem 'jasminerice', :git => 'https://github.com/bradphelan/jasminerice.git'
    gem 'rspec-rails'

    # Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
    gem 'jbuilder', '~> 1.2'

    group :doc do
      # bundle exec rake doc:rails generates the API under doc/api.
      gem 'sdoc', require: false
    end
  </code>
</pre>

The first thing to note is that this is a vanilla Rails Gemfile with a few exceptions. I removed the wide variety of comments that comes in a gemfile when you run `rails new` to clarify. There are two groups of three gems that we need for our application. The first is `jquery-rails`, `ember-rails`, and `'ember-source', '1.2.0'`. `jquery-rails` should be there by default. We'll need the [ember-rails](https://github.com/emberjs/ember-rails) gem which, as you can guess, installs Ember, Ember Data (Ember's data persistence library), and Handlebars (Ember's templating engine).

Next, we have our testing gems: `jasmine` and `jasminerice`. The `jasmine` gem gives us the Jasmine testing framework, and the `jasminerice` gem makes Jasmine play nicely with Rails's asset pipeline.

Now let's get those gems by running `bundle install` back at the trusty command line.

### Generators

Now that we've got those gems in our project, we'll need to run their generators, which will install a number of files for us automatically. Let's begin by running `jasminerice` and `jasmine`'s generators:

<pre>
  <code>
    rails g jasminerice:install
    rails g jasmine:install
  </code>
</pre>

The output should be a few create statements.

Now we'll need to get going with Ember's generators. As of this writing, Ember Data is at version `1.0.0-beta.4`. So let's use that, as opposed to the released `0.14` stable release; there have been a [number of breaking changes](https://github.com/emberjs/data/blob/master/CHANGELOG.md), which, if you're not prepared for, can cause a substantial amount of hair-pulling.

So let's run:

<pre>
  <code>
    rails generate ember:install --channel=beta -g --javascript-engine coffee
  </code>
</pre>

The `--channel` flag tells the gem which release channel to use; the options are `canary` (the master branch), `beta` (the current beta), or `release` (stable). We then tell the generator that we want to use CoffeeScript.

A few last bits of housekeeping. When we ran the Jasmine generator, it created an example spec for us that will immediately throw an error. Let's get rid of it:

    rm spec/javascripts/example_spec.js.coffee

Also, we removed the Turbolinks gem because it doesn't make sense to use that with an Ember project. So let's remove it from the `application.js` file. Now, let's rename our `application.js` file to `application.js.coffee` and make sure that it reads:

<pre>
  <code>
    #= require jquery
    #= require handlebars
    #= require ember
    #= require ember-data
    #= require_self
    #= require_tree .

    @App = Ember.Application.create()
  </code>
</pre>

This is pretty straightforward. We're just telling Rails's asset pipeline to compile all the files it needs. The only thing that might seem weird is our line beginning with `@App`. Calling `Ember.Application.create()` is what starts Ember's engines, but since we're in the global namespace, `@App` will complile down to `this.App`. In this context, `this` is `window`, which means that `App` will be an object available to the rest of our application, since everything has immediate access to window if it's running in the browser.

Now we can check our progress by starting our server with `rails s` and going to `localhost:3000/jasmine`. If we get a blank page (which we should, since we haven't written any specs yet), then we're in business! If you get any other errors that you can't figure out, please add them in the comments.

### Unit Tests

JavaScript model tests can be a great place to start in our testing suite because they can be quick to write and run, and they put us in a good place to think about our models' functionality and, therefore, our application's as a whole. From that vantage point, we can write descriptive and robust integration tests to spec our application's behavior.

For the purposes of this tutorial, let's begin to build an application for a bookseller to help calculate the revenue for their bookstore. A book will be our object under test, and properties of books that we'll want to use as the basis for our data model should start to come to mind: title, number of pages, price, etc. Here, we're going to jump right in and test a [computed property](http://emberjs.com/guides/object-model/computed-properties/), a core aspect of Ember's data model. A computed property is a function defined on an Ember object that is made up of two or more other properties. For example, if you have an object with a `firstName` property and a `lastName` property, you could define that object's `fullName` computed property function as:

<pre>
  <code>
    fullName: function(){
      return this.get('firstName') + ' ' + this.get('lastName');
    }.property('firstName', 'lastName')
  </code>
</pre>

Now, when you call `fullName` on your object, it will return that object's `firstName` property and `lastName` property, separated by a space. This approach to accessing these properties has two main benefits. Firstly, it's clearly shorter than writing the function body everywhere you want to access the combination of the two properties. But more importantly, it takes advantage of Ember's [two-way bindings](http://emberjs.com/guides/object-model/bindings/), meaning that when either `firstName` or `lastName` changes on this object, `fullName` will reflect that change.

So let's test an as-yet-undefined `revenue` function on our book that calculates the revenue of a particular book title. It will do so by multiplying the book's price by the units sold. Easy peasy.

Unfortunately, getting Ember, Jasmine, and Rails to play nicely together takes a little finagling. Let's find out why by beginning to write our model spec. Create a new file in the `spec/javascripts/models` directory (create that directory if it doesn't already exist) called `book_spec.js.coffee`.

Let's write this code, and we'll discuss it below: _Note: If this looks odd or wrong, keep reading._

<pre>
  <code>
    describe 'Book', ->
      describe 'revenue', ->
        it 'is a computed property of price and units ordered', ->
            book = (
              price: '13.00',
              unitsOrdered: '25'
            )
            expect(book.get('revenue')).toEqual 325
  </code>
</pre>

Okay. On the `Book` model, we're describing the `revenue` function, which is a computed property derived from multiplying the `price` by the `unitsOrdered`. Our expectation, then, is that when we set these properties and call `revenue` (via Ember's getter method) on the book, we'll get the product of those two properties. Now, here's where things get a little tricky.

We should get the following error: `Expected undefined to equal 325.` This is because the `revenue` method on `book` doesn't exist: we haven't written it yet. So let's write that method on our book model and see where that gets us.

Create a `app/assetes/javascripts/models` directory and add `book.js.coffee` to that directory. In that file, we're going to need to tell Ember what the properties of our model are, along with their data type, just like in Rails. So let's do that:

<pre>
  <code>
    App.Book = DS.Model.extend
      price: DS.attr('number')
      unitsOrdered: DS.attr('number')
  </code>
</pre>

We defined our `App` in `application.js.coffee`, and `extend`ing from `DS.Model` is what gives us the ability to access the values of our properties on the model. This is just like having a model that descends from Rails's `ActiveRecord::Base`. Now, we'll need to define our `revenue` computed property:

<pre>
  <code>
    revenue: (->
      @get('price') * @get('unitsOrdered')
    ).property('price', 'unitsOrdered')
  </code>
</pre>

If we run this test now, we'll still get the error `Expected undefined to equal 325.` This is because in our spec, `book` is just a POJO ([plain old JavaScript object](http://odetocode.com/blogs/scott/archive/2012/02/27/plain-old-javascript.aspx)). Ember doesn't know about it. We need to set up a mock data store, and tell Ember that `book` is an instance of `App.Book`. *Here's where it's going to get hairy, so hang in there.*

## The Hairy Part

We're going to need to create a mock data store to be able to persist objects' state for testing purposes. We're also going to have to take manual control of Ember's run loop.

### Aside

The run loop is where Ember's magic happens, and it's a series of events that deal with things like binding data and updating views when a model's property changes. It is, in essence, where Ember gets its power. As such, it's extremely complicated. But you can get a sense of it by reading Alex Matchneer's [Stack Overflow](http://stackoverflow.com/questions/13597869/what-is-ember-runloop-and-how-does-it-work) post, which goes into great detail.

For our purposes, however, feel free to think of it as a series of callbacks we're going to want to control.

### Back to the Hairy Part

In addition to defining a mock store, we're going to need to create a `container`, which is the [Ember module that handles dependencies](http://stackoverflow.com/a/14089090/931934). In our case, Ember is going to need to know that our declaration of `book` is indeed an instance of `App.Book`, our model. Lastly, as we discussed, we're going to have to take manual control of Ember's run loop so that we can call our computed property from within the loop, where the properties get computed (I think).

So let's look at our test now:

<pre>
  <code>
    describe 'Book', ->
      describe 'revenue', ->
        it 'is a computed property of price and units ordered', ->
          store = null
          App.Store = DS.Store.extend(
            adapter: App.ApplicationAdapter
          )
          container = new Ember.Container()
          container.register('model:book', App.Book)
          store = App.Store.create(
            container: container
          )
          Ember.testing = true
          Ember.run ->
            book = store.createRecord('book',
              price: '13.00',
              unitsOrdered: '25'
            )
            expect(book.get('revenue')).toEqual 325
  </code>
</pre>

What we've added is a mock data store, a container that treats `book` as `App.Book`, a call to put Ember in testing mode so we can control the run loop, and began the run loop so that Ember computes `book`'s property. This test will pass.

### Clean up

Congratulations. You've made it through the craziest part.

But looking back at our code, that test is really cluttered, and has a lot of code that deals with low-level aspects of Ember's internals that our test simply should not care about. Let's fix that. I'd like to fold that code into a place where all our specs will have access to it and, frankly, we don't have to look at it.

It turns out our `spec/javascripts/spec.js.coffee` file, which got created when we installed Jasmine, is a perfect place. Let's throw it in there. We'll have to make one modification to it, which is to make the `store` globally accessible. We'll do that in the same way we did with our call to `@App`: hang it off of the global `window` object. So our `spec.js.coffee` file should look like this:

<pre>
  <code>
    #= require application
    #= require_tree ./

    App.Store = DS.Store.extend(
      adapter: App.ApplicationAdapter
    )
    container = new Ember.Container()
    container.register('model:book', App.Book)
    @store = App.Store.create(
      container: container
    )
    Ember.testing = true
  </code>
</pre>

If you give that a scrutinizing look, you'll notice that we'll need to register every new model that we create with our `container`. I don't know a way around that. (If you do, let me know.)

But now, our spec looks like this:

<pre>
  <code>
    describe 'Book', ->
      describe 'revenue', ->
        it 'is a computed property of price and units ordered', ->
          Ember.run ->
            book = store.createRecord('book',
              price: '13.00',
              unitsOrdered: '25'
            )
            expect(book.get('revenue')).toEqual 325
  </code>
</pre>

This looks much more like a spec without any extraneous code (except for `Ember.run`, which is unavoidable.) And this spec should still pass.

## Conclusion

We're now well on our way to a robust test suite. As I said before, getting your testing framework set up is the hardest part of writing specs. The goal here was just to get Ember set up to work with Jasmine and Rails 4. There's a lot more that could be done, such as getting our tests to run on the command line so we don't have to open the browser. But the goal of this tutorial was just to get Ember playing nicely with Jasmine. If you had any problems along the way, please let me know in the comments.

Happy testing!
