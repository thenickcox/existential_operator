---
title: Listening to Backbone.Events
date: 2014-10-01 14:07 UTC
tags: backbone
---

I know I’m late to the party, but I’ve been digging into Backbone.js lately, and I’ve run across a few things that have really tripped me up because I didn’t understand the framework well enough.

In looking at the [documentation on listening to events](http://backbonejs.org/#Events-listenTo), for example, you find that `listenTo` does the following: “Tell an object to listen to a particular event on an other object.” Let’s take a look at an example. [READMORE]

    // Define model
    App.Models.Rapper = Backbone.Model.extend({});

    // Define View
    App.Views.RapperView = Backbone.View.extend({

      initialize: function(){
        // When the the model passed to this view's attributes change,
        // call `this.alertChanged`
        this.listenTo(this.model, 'change', this.alertChanged);
      },
      alertChanged: function(){
        console.log('model changed');
      }
    });

    // Instantiate rapper
    var snoop = new App.Models.Rapper({ name: 'Snoop Dogg', coast: 'West' });

    // Instantiate view and pass our rapper instance in
    var rapperView = new App.Views.RapperView({ model: snoop });

    // Change our rapper's attributes
    snoop.set({ name: 'Snoop Lion' });
    // logs 'model changed'


This only works as expected when we pass `listenTo` an instance of a model. This may seem self evident, but let’s take a look at what happens if you try to pass `listenTo` a class. _(Note: If this terminology seems wrong to you, you're right! Keep reading.)_

    // Define a movie model
    App.Models.Movie = Backbone.Model.extend();

    // Define a movie collection
    App.Collections.Movies = Backbone.Collection.extend({
      model: App.Models.Movie
    });

    // Define a movie view
    App.Views.MoviesView = Backbone.View.extend({

      initialize: function(){
                      // WRONG! We're listening to a class, not an instance
        this.listenTo(App.Collections.Movies, 'add', this.alertAdded);
      },

      // The function we'll eventually call when we successfully
      // listen to an instance's add callback
      alertAdded: function(){
        console.log('New movie added.');
      }
    });

    // Instantiate our collection and view
    App.movieLibrary = new App.Collections.Movies();
    App.movieList = new App.Views.MoviesView();

    // Create a new movie
    var movie = new App.Models.Movie({ title: 'Edward Scissorhands', year: 1990 });

    App.movieLibrary.add(movie)
    // Uncaught TypeError: undefined is not a function
    // Argh!

We get that most insanity-inducing and cryptic of errors: `undefined is not a function`. (We’ll take a look in Backbone’s source why exactly that happens later.)

In this case, it’s less obvious that you’d want to listen to an event on an instance rather than a class. Previously, we listened to an event on a model, and it makes sense intuitively that you’ll likely have more than one model, and you’ll want to listen to events on each. But it seems to me less likely that you’d have more than one collection, and here, I found myself confused when trying to pass that collection definition to the `listenTo` method on the view.

To fix it, let’s take another look at our view, and modify it by passing in an _instance_ of the collection.

    App.Views.MoviesView = Backbone.View.extend({
      initialize: function(){
                      // Listen to the instance we've defined
        this.listenTo(App.movieLibrary, 'add', this.alertAdded);
      },
      alertAdded: function(){
        console.log('New movie added.');
      }
    });

Now, let’s add that movie to the collection.

    App.movieLibrary = new App.Collections.Movies();
    App.movieList = new App.Views.MoviesView();
    var movie = new App.Models.Movie({ title: 'Edward Scissorhands', year: 1987 });
    App.movieLibrary.add(movie);
    /* logs 'New movie added.' */

That logs our message as we expect. It’s also notable that when we instantiate our objects, we hang them off of the global `App` object. If we had instantiated it with a normal variable, like `var movieLibrary = App.collections.Movies()`, then it’s not defined when we call it earlier in the file. Defining it on the `App` object makes it available in our `App.MoviesView` that we wrote earlier. This enables us to write our app in a way that makes sense, rather than based on what variables are available at what time.

This was confusing to me until I looked at the output of the following two statements:

    console.log(typeof App.Collections.Movies)
    // 'function'
    console.log(typeof App.movieLibrary)
    // 'object'

Of course. `App.Collections.Movies` isn’t actually an object (or a prototype, as I referred to it earlier), it’s a constructor function. I got so caught up in looking at its attributes that I looked right past the fact that we were just passing an object to its `extend()` method. In this sense, the definition in the docs of what the `listenTo` function does.

I thought it might be helpful to look into Backbone’s source and see what exactly the error was. Let’s take a look at what’s happening internally.

Because an object can listen to events on multiple objects, the listening object has to have some way of keeping track of the objects its listening to. Here’s the implementation, taken straight from Backbone’s source.

    var listenMethods = {listenTo: 'on', listenToOnce: 'once'};

    _.each(listenMethods, function(implementation, method) {
      /*
         When we come in here with the `listenTo: ‘on’`
         key-value pair, `listenTo` is the implementation,
         and `on` is the method.

         Events is Backbone.Events, an object that has several
         functions, such as our `on`, `off`, `trigger`, etc. It
         gets mixed into our views.  */
      Events[method] = function(obj, name, callback) {
        /*
          Here’s where it gets interesting. In this function,
          we have 3 variables. `obj`, `name`, and `callback`.
          The important one for our purposes is `obj`. When
          we tried to listenTo an event on our constructor
          function, `obj` was actually a function. When we
          made it an instance of that collection, it is an
          object. Let’s see what happens.

          These three lines give a method called `_listeningTo`
          to the object (or, erroneously, function) we want to be
          listening. It then gives the object we’re listening
          to a unique id so we can keeping track of it. The
          underscore library has a simple method that just
          increments its uniqueId property each time it’s called.
          It gets called every time an object is instantiated.
          The `l` here is a prefix, and stands for listen. This
          is to tell the types of things we’re giving ids apart.
          For example, Backbone models are created with an id
          using this _.uniqueId method and prefixed with a ‘c’.  */
        var listeningTo = this._listeningTo || (Thiss._listeningTo = {});
        var id = obj._listenId || (obj._listenId = _.uniqueId('l'));
        listeningTo[id] = obj;

        /*
          When we get here, `callback` is our logging function
          and name is `on`.  */
        if (!callback && typeof name === 'object') callback = this;

        /*
          And here’s where we fail. If `obj` is truly an object
          when we arrive here, we can access a method property on
          it with bracket notation (if the property contains a
          function) and apply it to our three variables. If we
          have a function instead, we’re going to get an error
          when we try to call a method on it. Thus, TypeError:
          undefined is not a function.  */
        obj[implementation](name, callback, this);
        return this;
        };
      });
    }

There you have it. I was hesitant to dig into Backbone’s internals in my first draft, but I really wanted to know why we were getting that error. On the way, we illuminated how Backbone implements keeping track of which objects another object is listening to. Fun!





