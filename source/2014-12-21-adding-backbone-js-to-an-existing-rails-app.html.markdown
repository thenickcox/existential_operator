---
title: Adding Backbone.js to an Existing Rails App
date: 2014-12-21 02:33 UTC
tags: backbone, rails
layout: post
---



_Editor’s note: This tutorial assumes a basic comfortability with basic Rails concepts, as well as running tasks on the command line. An understanding of jQuery is helpful, and while CoffeeScript knowledge isn’t absolutely mandatory, not having a strong aversion to it would be beneficial._

Since the dawn of the rise of front-end JavaScript frameworks, much of the user interface on modern websites has been handled on the client. Single-page applications have become widespread, and users have come to expect a richer, more desktop-like experience. Increasingly, rendering templates on the server and re-rendering the whole page with each request has come to feel slow.

[READMORE]

Web frameworks like Rails can make developing API-driven JavaScript-heavy applications much easier, due in part to the efficiency of the asset pipeline and Rails’ excellent out-of-the-box support for JSON, consumed by the front-end application.

Front-end frameworks like Ember.js and Angular can be excellent solutions, and work quite well on greenfield Rails applications. However, if you’re building onto a legacy application and need an AJAX-heavy feature, you may not want the cognitive overhead of a heavyweight MVC library like Ember or Angular, to say nothing of the added page weight that libraries like these can add, which may not be worth the overhead if you’re only implementing a small feature set.

In the context of adding a JavaScript UI to a legacy Rails app, Backbone shines. It is lightweight and can provide a cogent and maintainable architecture for your client side JavaScript code.

In this tutorial, we’ll take a look at adding Backbone to an existing Rails application. We’ll start with an extremely simple REST-style application, and look at setting up the Backbone app, basic code organization, and a few gotchas that you’ll run into when integrating Backbone into your Rails app. We’ll also look at how design patterns like the single responsibility principle apply to modern JavaScript architecture.

## Step 0: Get the code ##

I’ve created a repository on GitHub that reflects the application in its starting state, before any of the Backbone code is added. To get it set up, start by cloning [the repository](https://github.com/thenickcox/backbone-rails-demo).

_Note: If you want to follow along step by step, once inside the directory (`cd backbone-rails-demo`), checkout the `step_0` branch (`git checkout -b your_branch step_0`). That will give you the starting point for this application. If you get confused at any point, stash your changes by running `git stash` and checkout the branch of the completed next step, such as `git checkout step_1`. That will give you the code at the next save point that you can compare against yours._

Once you’ve cloned the application and checked out the appropriate branch, run `bundle install`. From there, you can follow the directions on the README. (If you run into any issues getting your app started, please [file an issue](https://github.com/thenickcox/backbone-rails-demo/issues) and I’ll do my best to get it resolved. Nothing’s more frustrating than not being able to get your app set up!)

Here’s a brief overview of the Rails app: It is a CRUD app for cataloguing a record collection. I’ve created some seed data with some albums for a user. (Load the seed data by running `rake db:seed` from the directory your app is in.) The seed data is full of classic jazz records.

There are two models in this app: users and albums. You might have already guessed that the relationship between these models is that a user `has_many :albums`. Go ahead, fire up the application and add a few of your favorite albums.

Here is the feature we’re going to build with Backbone.js: the `albums` table has a `featured` boolean column. This way, users can feature their favorite albums of their collections for the world to see. Currently, to unset this field, the user has to go to the `edit` view for the album. The requested feature is that a user can set the `featured` flag on their album from the edit page and have it show up in a display box above the table showing all of their albums. This box should display an unordered list of the featured album names, the link should go to the `show` page for the album, and there should be a button labeled `unfeature` that will remove the album from the list. If there are no featured albums (such as in the case where the last featured album has been removed from the list), the box should display the text, “There are no featured albums.”

This seems quite simple, but it will take a good bit of code to get this set up. So without further ado, let’s jump in.


## Step 1: Add necessary gems to set up Bootstrap.

From the root of your Rails application, open up your `Gemfile`.  Add two gems to the bottom:

<script src="https://gist.github.com/thenickcox/3dadcb3cbca7513e8d4a.js">
</script>

The first gem will give us both Backbone.js and its dependency, Underscore.js, vendored for the asset pipeline. (Backbone’s other dependency, jQuery, is already included in the Rails asset pipeline by default.) Underscore is a JavaScript library that adds a lot of great utility methods to the JavaScript core that Backbone uses heavily. We’ll be using its `template` method in our Backbone Views.

The second gem gives us haml.js, a templating library for our JavaScript templates. haml.js is great because it gives a haml-like syntax for JavaScript templates. However, it’s important to note that this gem isn’t currently under heavy development, so its support for features and bugfixes may be lackluster in the future. However, I tried a number of gems that included templating libraries in writing this tutorial, and this was by far the easiest to get up and running with.

To install the gems, run `bundle install`. `backbone-on-rails` also has a custom installer that we’ll run with `rails generate backbone:install`. Don’t forget to restart your server.

Refresh your application. If a JavaScript alert shows with the text, “Hello from Backbone!”, everything has been installed correctly.

By default, `backbone-on-rails` names our Backbone app a camel-cased version of the name of our Rails application, which in this case results in `BackboneRailsDemo`. We’re going to rename it to `App` to follow the community conventions. We’re also going to remove the code that it placed in the `app/assets/backbone_rails_demo.js.coffee` file. In the `app/assets/application.js file, replace 

<script src="https://gist.github.com/thenickcox/8d56e3ec2e1879f4cbff.js">
</script>

with

<script src="https://gist.github.com/thenickcox/8b380b88f7ab8b4d0012.js">
</script>

Also, add an `App` object, where we’ll namespace three of the four major components of Backbone: models, collections, and views. (The fourth is the router, but we’ll talk about that later.)

<script src="https://gist.github.com/thenickcox/daacdaa1895e5d220195.js">
</script>

Calling `require self` above requires the `application.js` file, at the bottom of which we’ve placed our global `App` object. With that object, we’re effectively namespacing our `Models`, `Collections`, and `Views` objects. This will come into play later.

Save this and refresh the page. Open up the JavaScript console, and type `Backbone` and hit enter. You should see something like the following:

<script src="https://gist.github.com/thenickcox/a3d7f0c6f1f660fb80ed.js">
</script>

That’s the global `Backbone` object. We’ll talk about some useful features of that in a bit. Now type `App` and hit enter. You should see:

<script src="https://gist.github.com/thenickcox/ccae5a51144f37b40ae6.js"></script>
</script>

Cool. That’s the object we just created in our `application.js` file. It’s the same object under which we namespaced our models, collections, and views, which you can see inside our object in the console. Lastly, delete the `*.jbuilder` files that were created by the scaffolding. And that’s the end of step 1.

## Step 2: Model Data on the Client ##

One of my hopes with this article is to show a refactor process, rather than skipping steps to show the end result. To that end, I want to look at building the app in one file so we can see the parts working together, and only then breaking the models, controllers, and other components out into separate files for the sake of maintainability.

Before we write any code, let’s take a second to think about the implementation specifics of our feature. In short, we’re going to be toggling the `featured` property of our Rails model. This modeling will be mirrored on the client through Backbone: that is to say, we’ll create a Backbone model that is a JavaScript representation of the JSON response our Rails server gives us when we hit the API endpoint. It’ll have all the same attributes. We’re also going to be responding to a user’s click event to set that property, which will be handled by the Backbone view. Our view will then point to a template, in which we’ll place the markup that we want to place in the DOM. Our model will be part of an array of models (rendered as a list), which Backbone considers a collection. This list will have its own view, since we’ll have to do things like display a spinner while the models are loading, and display text that tells the user that there are no featured albums when the list is empty. These will be the responsibility of the collection view.

At the bottom of our `application.js` file, let’s add the following:

<script src="https://gist.github.com/thenickcox/9cb345f506a88391f162.js">
</script>

Now we can create the `app/assets/javascripts/app.js.coffee` file that our `application.js` manifest is including. In it, let’s begin with our model:

<script src="https://gist.github.com/thenickcox/52aefaff03ae976406cb.js">
</script>

We’ve now created a Backbone model. Like a Rails model, this object will act as a class that will define the attributes we can set and access on instances of that object. As you can see, we’ve placed our `Album` in the `App.Models` namespace, which we set up above. If we refresh the page and open the JavaScript console, we can create a new instance of that `Album` model. Because we haven’t defined the attributes of the model and synced it with the API, we can define arbitrary attributes that we can access and write through Backbone’s getter and setter methods, conveniently named `get` and `set`, respectively. Let’s test this in the console:

<script src="https://gist.github.com/thenickcox/e01b64e2061a30ba2e4a.js">
</script>

This is the API we’ll use to set the albums’ `featured` flag to false through the view. But we’ll get to that.

First, we’ll need to create a collection that will contain the models we’re creating. We’ll also need to give it an API endpoint to tell it where to look for models. One thing that we know for sure is that an AJAX `GET` request that hits the API should only return the albums for the user whose index page we’re looking at. So let’s modify the API response in the `index` method of the albums controller.

<script src="https://gist.github.com/thenickcox/4b5ea042151489a11545.js">
</script>

Great. Now if we create a collection of Backbone albums and point it at that API endpoint, we should get back those albums. Let’s build that collection now.

We’ll just need to tell it which model to use and a URL from which to fetch new records. For now, let’s hardcode that URL to use the user id of the user created by the database seeds.

<script src="https://gist.github.com/thenickcox/6643c1f488f21954a2ac.js">
</script>

Now if we go back to the browser and refresh, we should be able to fetch those records from the API we created in the controller by instantiating this collection and using `Backbone.Collection`’s fetch method:

<script src="https://gist.github.com/thenickcox/e372d23362814eddb724.js">
</script>

Okay. We’re definitely making progress. (We’ll come back to making the user id in the url dynamic.) We’ve fetched the albums from the API and can access their attributes. That foreshadows what we’re going to do with our Backbone list view: loop through the models and instantiate their corresponding views, passing in the models, whose attributes we’ll access in our templates, which will in turn render onto the screen.

In the next step, let’s render the list view. This part is pretty complicated, so we’ll break it down into small steps. It also includes some key concepts in Backbone, so I’d recommend reading and re-reading this section before implementing the code. _(Note: If anything at all is unclear here, let me know by filing a GitHub issue. Understanding these concepts is key to developing Backbone applications, so if something needs a clearer explanation, that’s a bug!)_

## Step 3: Rendering the Item View ##

We’re going to create an element in our albums index view for the list view to render into. For now, it’ll be an empty element. Then, when the JavaScript is evaluated, it’ll populate that element with our model data. So let’s place this above the index table:

<script src="https://gist.github.com/thenickcox/009a72304f410b0589c2.js">
</script>

Great. Now for the fun part. Let’s create our item view. This view will be in charge of rendering the attributes of the model into the browser, as well as handling any events that happen in its DOM context, such as clicks. (Hang in there, that’s a later step.)

Under the hood, all of these `Backbone.[Component].extend` calls are constructor functions. Their job, when called, is to create an object. So, we’ll be instantiating our views with the help of these constructor functions.

By default, Backbone views render into a `<div>` element, but since we know we want an `<li>` element, we can declare that by calling the view’s `tagName` method.

<script src="https://gist.github.com/thenickcox/5c65d25e2ef06332efa5.js">
</script>

Okay. Given a model with attributes, what are we going to want to print onto the screen? Since our data model is an album, let’s start simple by just printing its title. We’re going to need to tell Backbone how to create the markup for our model, given some attributes, and we do this with the `template` method. We’ll make use of the Underscore library’s own `template` method, which takes a string with hooks for JavaScript evaluation, and, given a model with some attributes, fills in those dynamic placeholders. Let’s see that in action:

<script src="https://gist.github.com/thenickcox/ff186f4d8ff5bb0047af.js">
</script>

You might recognize that angle bracket syntax from `ERB`, if you’re used to working with Rails. If you pass an object called `myObject` with a title attribute, this will evaluate `myObject.title`. Let’s write a render method for this view that takes a model (which we’ll pass in via the view’s `initialize` method for now) and place it in this template, given its attributes. In that render call, we’re going to fill in the HTML `el` of this view. The `el` is simply the view’s DOM context. (Think of it as the HTML el-ement we’re rendering into.) Into that `el`, we’ll render the content of this template, to which we’ll pass the model’s attributes (and it will access the `title` attribute). That looks like this:

<script src="https://gist.github.com/thenickcox/cd68c56bf98fe10943db.js">
</script>

Two quick things to note about that `render` method. That lonely `this` on the bottom line will compile down to the following JavaScript: `return this`. Effectively, that will return the item view object, which will allow you to call another of its methods. We’ll see where this comes in handy in a minute.

The second thing is that foreign-looking `@$el`. This is a CoffeeScript-specific syntax which  which is essentially just a jQuery-wrapped version of its `el`. This will also come in handy shortly.

Now let’s go back to the browser and refresh. We shouldn’t see any errors. In the console, let’s instantiate that view:

<script src="https://gist.github.com/thenickcox/4b77b5ba37bb985246ff.js">
</script>

There, we can see our view’s `$el` as well as its `el`. (Notice, like I mentioned, that the `$el` is a jQuery object.) But that didn’t change the output on the screen. That’s because we didn’t add it to the DOM. Let’s do that now. We can simply append this `li`  to our `ul` with an id of ‘featured’. Let’s do this in the console:

<script src="https://gist.github.com/thenickcox/132605a707cb5f76bea0.js">
</script>

When you press return, you should see that `li` show up in the `ul`. Exciting! Remember when I mentioned that those two bits would come in a handy? Well, the first one has: we called `render()` on the view, and since we returned `this`, meaning the instance of the view we’re working with, we were able to chain the `el` onto the `render()` call. And now the `el` is populated with the contents of our template, which is why there was content in it when we rendered it onto the screen.

We didn’t write a lot of code in this step, but we’re really starting to see the power of Backbone! In our next step, we’ll grab our models from the server and render them into the list view.

## Step 4: Rendering the List View ##

Think of the list view as a container for item views. Indeed, this is reflected in our markup: our list view is a `<ul>` element, and the items will be rendered as `<li>` elements.

Let’s create our list view. Since we already have an element into which we’re going to render it, we can declare that explicitly in the view. The `el` property takes a jQuery selector:

<script src="https://gist.github.com/thenickcox/b268197760c1537f0837.js">
</script>

We’re going to instantiate this list view, and it’s going to need access to a collection: our collection of albums. We can think of this view’s `render` method as looping through the models of the collection, instantiating item views for each of them, passing in their attributes (like we did in the console at the end of the last step), and appending them to the item view’s `el`. That looks like this:

<script src="https://gist.github.com/thenickcox/d320e0b87fd157ad7689.js">
</script>

That bottom line should look familiar: that’s what we just did in the console at the end of the previous step. You may be asking yourself where `@collection` (or `this.collection`) came from. Well, we’ll be passing that collection in when we instantiate the list view. Let’s do that using jQuery’s `document.ready()` method, beautifully abbreviated in CoffeeScript like so: `$ ->`.

When we instantiate our view, we’ll need to populate it with data from our collection. Let’s instantiate our collection and call Backbone.Collection’s [`fetch()` method](http://backbonejs.org/#Collection-fetch), which will retrieve the models from the server at the URL we’ve set for it (found on its corresponding model).

<script src="https://gist.github.com/thenickcox/c42fbdd1838f4e32eda3.js">
</script>

Refresh the page. Huh. Nothing happens! Well, let’s take a look at that `App.albums` collection by logging it to the console:

<script src="https://gist.github.com/thenickcox/386d44ea96088ff3b7ff.js">
</script>

Why didn’t anything happen on the screen? Well, we didn’t tell our list view to render! Before we look at why, let’s take a conceptual look at how we’ll be doing that.

You’ll often hear about how Backbone gives structure to your applications. One way it does this is enforcing the principle of [information hiding](http://en.wikipedia.org/wiki/Information_hiding). It shouldn’t be our collection’s responsibility, for example, to render a view. (The more “knowledge” an object has, the harder it becomes to maintain.) In the case of the collection we’ve just `fetch()`ed, Backbone makes use of the [pub/sub](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) design pattern, short for publish-subscribe. When the collection `fetch()`es the records from the server, it broadcasts, or “publishes” a `sync` event. We can tune into, or “subscribe” to, this event from our view. This way, Backbone lends itself to a strong [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns): I, as the collection, have just synced. You, as the view, can do whatever you want with that information, but I’ll stay in charge of my domain: organizing my array of models. If you want to render, that’s your business. Because of this, our code will be loosely coupled and easy to change: If we change the URL we’re fetching from or the scope of the models, our view won’t change. Likewise, if we render into another element, or modify our template, our collection won’t change. This is the key to maintainable software.

Now, let’s tell our view to subscribe, or `listenTo`, the `sync` event on the collection. When the collection publishes the `sync` event, we’ll call the view’s `render()` method. If we think about what will happen when our view renders, we’ll basically be looping through each item view and appending that to the list view, or appending `<li>`s to the `<ul>`. One last note: we’re going to place our `listenTo` call inside of the `initialize` method so that our view begins to listen when it’s initialized. In order to do this, we have to pass in the collection to which it’s listening to its constructor function. All together, that’ll look like this:

<script src="https://gist.github.com/thenickcox/0a2fb302811f313888b8.js">
</script>

If we delete our `initialize` method from the model, which was just statically setting the `title` attribute to “Giant Steps” and refresh, we’ll see a list of the title’s of our albums!

Let’s look at this again, because this communication between objects is central to Backbone development. Starting on the `$(document).ready()` call, we’re instantiating our list view and passing it the `App.albums` collection we’ve instantiated on the line above. This will call the `initialize` method on the list view, which will set it up to listen to the collection. Back at the bottom, when we call `App.albums.fetch()`, that’ll trigger a `sync` event (you can see that in the Backbone source [here](https://github.com/jashkenas/backbone/blob/master/backbone.js#L455-L467)), to which our list view will respond by calling `render`.

And that’s it! We’re now dynamically rendering our models, which are fetched from the server with an AJAX call, into our view! But shouldn’t we be only grabbing the models that are actually featured? Glad you asked.

## Step 5: Scoping the Collection ##

One of the really helpful things about using Backbone with a Rails project is that Backbone provides a familiar API for scoping models. Because Backbone.Collection has a `where` method that takes a hash, the Rails developer can feel comfortable with a syntax similar to ActiveRecord’s `where` method. Let’s try this in the console:

<script src="https://gist.github.com/thenickcox/5cd426a392c7904e83ab.js">
</script>

And if we click the relevant arrows, we get something like this:

<figure>
  <img src="/images/love_supreme_console.png" />
  <figcaption>Our console output</figcaption>
</figure>

Great. Now we can imagine what it might look like to scope that to the featured albums. But first, we’ll have to go back to our Rails app and create a migration for that attribute.

We’re going to want to add a boolean column to the `albums` table, and it should default to false. Let’s write the migration for that. I like to use Rails’s migration generator to generate my migrations so they have the correct timestamp. Let’s run `rails g migration add_featured_to_albums featured:boolean` from the console, then open it in our text editor and add the default value:

<script src="https://gist.github.com/thenickcox/f1f644cdb25f189bc7cb.js">
</script>

Now let’s run `rake db:migrate` from the console, restart our rails server, and we should be set.

When we go back to the browser, let’s try to find any featured albums:

<script src="https://gist.github.com/thenickcox/94b8035193a584bdd137.js">
</script>

Empty array. That makes sense, because we defaulted the column to false, and we haven’t set any to featured yet. Let’s do that from the Rails console now:

<script src="https://gist.github.com/thenickcox/9b002d856209b67a1a1e.js">
</script>

Now if we go back to the browser and run our previous console function, we should get back that first album. And indeed we do:

<figure>
  <img src="/images/love_supreme_console2.png" />
  <figcaption>Our console output</figcaption>
</figure>

Great. Now all we need to do is tell our collection to use that scope in our `app.js.coffee` file. We’ll create a `featuredAlbums` method on our collection that returns that scope, and then use that in our render function instead of the whole collection:

<script src="https://gist.github.com/thenickcox/2ef9b0de24cf2541f9d5.js">
</script>

When I was writing this, I initially forgot to add the parentheses in the render call at `@collection.featured()`. Without those, the `featured` method on our collection doesn’t get called, so don’t forget them!

Now let’s refresh the page, and we should see the title of just that one album. Great! That’s the end of step 5.

## Step 6: Scope Collections to Current User ##

Looking at our collection with the hardcoded url for fetching albums reminded me that we need to scope that to our current user. We could write an automated test for that, and we would in the real world, but the focus for this tutorial is to get up and running with Backbone, so let’s test it manually for now.

I’ve been signing in with the admin user created by the seed file: admin@test.com, passw0rd. That user is the User with an `id` attribute of 2. Since we haven’t placed any authorization rules on the index page for these users (again, not the focus here), we’re currently able to go from `http://localhost:3000/users/2/albums` to `http://localhost:3000/users/1/albums` and see the same album title in our featured spot. Definitely not what we want.

We need to tell our collection to get only the featured albums for the current user. In our case, that’s the user with the `id` of 2. There are a number of ways to do this. When I first wrote the feature for my side project that this tutorial is based on, I implemented this by splitting the URL on the slashes and getting the number: `window.location.pathname.split('/')[2]`. But that’s a pretty brittle implementation. For example, what if the URL structure changes? When I first refactored the feature, I placed a JavaScript tag at the bottom of the view and hung a variable off of our `App` object that got the `user_id` from the params:

<script src="https://gist.github.com/thenickcox/34e1fbf24b7dab6967cd.js">
</script>

This is slightly better, but it unnecessarily places JavaScript in our view, which is avoidable. It also interpolates a Ruby string inside a piece of JavaScript code inside of haml view: nearly every language in our application is colliding here. And while sometimes that can seem useful (and even fun!), that could lead to some unnecessary cognitive overhead down the road. So let’s avoid that here.

I was reviewing a feature at work written by one of my colleagues, and I noticed a clever implementation of a solution to this problem. In the view, he placed the user id in a `data` attribute on a DOM element, then accessed that from the JavaScript app. This is by far a better solution because it doesn’t mix languages in the view, and it keeps a proper separation of concerns in our app.

Let’s add a data attribute with the `user_id` from the params to our featured ul in `app/views/albums/index.html.haml`:

<script src="https://gist.github.com/thenickcox/29c0286a359c88fc6fb6.js">
</script>

Now we can access that from the `url` method in our collection:

<script src="https://gist.github.com/thenickcox/df087802f40fe9ea89a7.js">
</script>

Now when we refresh, we should see the same featured album, but when we go to `http://localhost:3000/users/1/albums`, we should see the album list empty.

Now onto step 7!

## Step 7: Add Link to Show and Unfeature Button ##

We’re printing out the names of the featured albums in our `div`, and while that’s a great start, it’s not all that “interactive.” Let’s add a link to the `show` action of the controller and a button to unfeature the album. Essentially, then, we’ll be writing our template.

Since we want to go to the show page, our url should be `/users/2/albums/1`, where the first integer is the user id, and the second is the album id. That’s simple enough to create out of an Underscore template, since both `user_id` and `id` are present in the attributes hash in our Backbone model, based on the JSON response from our Rails app. Let’s do that now:

<script src="https://gist.github.com/thenickcox/88e6653264c3aa525bbd.js">
</script>

Great! Let’s refresh the page. That should now be a link. If you click on it, it should go to the show page for that album.

## Gotcha: Chrome JSON issue ##

If you’re developing this in Chrome and you go to the show page for the album, then click back, you’ll likely see a string of raw JSON rendered in your browser instead of the index page. That’s because, in Chrome’s opinion, the last request you made was to `albums.json`, not the show page, so it’s rendering the JSON string. It’s a perplexing bug, and one I spent a lot of time trying to figure out. If you’re interested, [here](http://stackoverflow.com/questions/26531024/rails-controller-renders-json-in-browser)’s the StackOverflow question I asked and some research around it. If not, the TL;DR is that this tells the browser that two cacheable requests from the same URL with different file extensions should be treated differently. So add this to your `ApplicationController`:

<script src="https://gist.github.com/thenickcox/a5ee516fa899ae890737.js">
</script>

## Step 7 (continued) ##

Okay. Now that we’ve got that pesky Chrome issue out of the way, let’s add a button. We’ll just add a class of ‘unfeature’ so that we can listen to its click event:

<script src="https://gist.github.com/thenickcox/5d8da248ed865dbd93e7.js">
</script>

With the `&nbsp;&times;`, we’re just adding a non-breaking space (just adding a space wouldn’t create any separation between the words), and a `times` symbol, which is essentially an `x`, but looks more like the “delete” symbols we’re used to seeing than simply an x.

_Note: If this template looks out of control to you, hang in there! We’re going to refactor it._

Now if you refresh, everything’s there, but it just doesn’t look right. I just added some quick CSS to smooth that over a bit:

<script src="https://gist.github.com/thenickcox/25809bfbe7fca2bd2a73.js">
</script>

In order to do this, I renamed my `application.css.scss` file to `application.css.sass` (because I hate typing curly braces when I don’t have to!) and added a file called `app/assets/stylesheets/albums.css.sass`, where I put the above CSS.

Okay. We’re looking at two issues right now. First, our button doesn’t do anything! We’ll take care of that in the next step. But we also have this pretty nasty template just hanging out in our view. We’ll take care of _that_ when we refactor our large app into discrete files. Hang in there.

## Step 8: Handle Button’s Click Event ##

The Backbone.View module mixes in the [Backbone.Events module](http://backbonejs.org/docs/backbone.html#section-16), giving you the ability to bind to a number of preconfigured events in your template. In this case, it’ll probably help to see an illustration.

In our `AlbumItemView`, let’s add an `events` property. This takes a hash, or a series of key-value pairs in the following format:

<script src="https://gist.github.com/thenickcox/c90c243a12d6c52d8069.js">
</script>

So ours is going to look like this:

<script src="https://gist.github.com/thenickcox/bfe2f0a52310a7312156.js">
</script>

Thus, when the DOM element (inside of this view’s `el`) with the class of `unfeature` receives a `click` event, the `unfeature` method in this object will be called. Let’s write the simplest possible `unfeature` method to make sure it’s getting called properly. Your whole view should now look like this:

<script src="https://gist.github.com/thenickcox/b3e7247da686e75cdf72.js">
</script>

If you click our unfeature button, you should now see `clicked` in the JavaScript console. Great.

So now, what should that `unfeature` method really do? Well, it should set the `featured` property on this view’s model to false and save it, and then remove that `li` from the DOM.

Recall that, inside of our list view, we’re passing our model into our item view on instantiation:

<script src="https://gist.github.com/thenickcox/aa55668346476b53321c.js">
</script>

So now we always have access to this view’s model inside of the view by calling `this.model`, or in CoffeeScript, `@model`. We’re going to set its `featured` property to false with Backbone’s `set` method, which takes a model property and a value. Let’s put that in our `unfeature` method now:

<script src="https://gist.github.com/thenickcox/f86da0ab5655b195170a.js">
</script>

Cool. Now refresh the page and click that unfeature button. Nothing happens. Why not? To find out, open the JavaScript console and refresh the page. Let’s find that album in the console:

<script src="https://gist.github.com/thenickcox/c926f02f92918cb9b1e2.js">
</script>

_(NB: `Backbone.Collection`’s `findWhere` method is just like the `where` method we used before, but it just returns the first model that matches the passed-in hash.)_ Now that we’ve got our model, let’s set its featured property to a variable so we can see if it’s changing. We’ll do this with the complement to `Backbone.Model`’s `set` method, `get`:

<script src="https://gist.github.com/thenickcox/9b2af66dfb4aba42f8b9.js">
</script>

Now click the unfeature button and run `console.log(album.get('featured'));` again. You should get `false`. But if you refresh the page and log the same property to the console, you’ll get `true`. Have you already figured out the problem? That’s right, we’re marking the `featured` property as false, but we’re not saving the model! Lucky for us, it turns out that `Backbone.Model` offers us a `save` method with the same method signature as `set`, meaning it takes a hash of values that you want to save on the object. So let’s change `set` to `save`:

<script src="https://gist.github.com/thenickcox/a14267ba8654cdef5395.js">
</script>

Now refresh and click the unfeature button. Now let’s open up our console and see if that worked:

<script src="https://gist.github.com/thenickcox/7ad8b3f203813ac3d870.js">
</script>

Now you should see false. If you refresh again, it doesn’t show up! That’s great, but in order to keep testing this, we’ll have to set it back to featured in the rails console. Open a new tab in your terminal in your project directory, and start the console:

<script src="https://gist.github.com/thenickcox/378a7573db0bc0b3a776.js">
</script>

Great. Now let’s reset that `featured` property:

<script src="https://gist.github.com/thenickcox/89b0435a867c8abe754e.js">
</script>

Now if we refresh, the album should show up again. This time, we need to remove that `li` from the DOM when we click that button. That’s as easy as adding a call to `this.remove()` in the view, shortened in CoffeeScript syntax to `@remove()`

<script src="https://gist.github.com/thenickcox/512b0b0f6159ca9bc4aa.js">
</script>

Okay. Now refresh and click on the unfeature button. It should remove that `li` from the document, and if you refresh, it doesn’t show up! That’s pretty exciting. Everything is working as expected!

Well, almost. Now you’ve got this nagging feeling that when there are no featured albums, it’s just a huge white box. Let’s commit what we have and fix that in the next step.

## Step 9: Empty Placeholder Text ##

In this step, it’ll be helpful to save our console query to reset the `featured` property to `true`, since we’ll be testing our changes. That way, we can just press the up arrow in the console and return, and it’ll update our model back to the featured state so we can see it in the browser again.

We’re going to need check the number of `featured` albums in our view. But there are a couple of ways to do this. We could check the number of `li` items in our `#featured` div, or we could directly ask our collection how many featured albums it has. Consider that the responsibility of the Backbone view is to update the DOM in response to changes in the data model as well as handle any user interaction. In this case, the albums collection shouldn’t be concerned with the UI state.  Thus, if we confine the UI-based logic to our view, it’ll ensure a good [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns).

Because we’ll check for the number of items after we unfeature one, that same `unfeature` method seems like a logical place to put that view logic. Let’s try a first draft of the method. In essence, we’re going to want to check the number of `li` elements in the `ul` with the id of `featured` (the same `ul` that acts as the `el` for the AlbumsListView). If it’s 0, then we should show some placeholder text (by appending it to the parent `div`). Here again, CoffeeScript’s English-like syntax shines:

<script src="https://gist.github.com/thenickcox/4ad2cc1302e0925f93c7.js">
</script>

If we go back to our console and set the album back to the featured state (see above) and refresh the page, we’ll see that, as expected, after we click that “unfeature” button, the placeholder text shows up. But we have two new problems: that `unfeature` method looks really ugly with a huge line of logic at the bottom, and if we refresh, there is neither a featured album nor placeholder text.

Let’s start by placing that logic in a separate method and calling that after we call `@remove()`:

<script src="https://gist.github.com/thenickcox/835962ed4e17c470b88a.js">
</script>

Okay. This looks a little better. That way, the `unfeature` method doesn’t have to be concerned with the number of `li` items there are; it’s not its responsibility. It simply has to call another method.

But should this method be the responsibility of the album item view? According to the [Single Responsibility Principle](http://robots.thoughtbot.com/back-to-basics-solid), classes (or JavaScript objects) should be as small as possible while performing a useful contribution to the application. One helpful exercise is to try to articulate out loud what responsibility you think your class has. (If you find yourself saying “and,” that’s often, though not always, an indication your class is doing too much.) Currently, our item view creates a visual representation of our data model, which is a single instance of an album, but it also responds to changes in the collection by updating the DOM in response to its emptying. That doesn’t seem right. Let’s see if we can narrow the scope of our item view to simply rendering and updating the page according to updates in the model.

In order to do that, we’ll have to place our `setPlaceholder` method in the list view, where it more naturally belongs. Go with me here for a minute. Previously, we used `Backbone.Event`’s `listenTo` method in the list view to respond to changes in the album collection. The collection didn’t tell the list view what to do, it just told it that it had synced, and left it to the list view to behave accordingly. That’s all captured here:

<script src="https://gist.github.com/thenickcox/f9e7b319bf3012b9ae9c.js">
</script>

We can use a similar technique to pass a message from the item view to the list view. Here, we after we `remove()` the item view’s `$el` from the DOM when the model is unfeatured, we’ll tell the list view that we’ve done so, and leave it to the list view to check if it needs the placeholder text. Let’s begin by writing the code we’d love to write, and work backward from there. So here’s what I’d love for our unfeature method to look like:

<script src="https://gist.github.com/thenickcox/0662795430c16f27f9c8.js">
</script>

We already had the first two lines. I added the last line to say, when we remove an element, this should trigger a method on the list view to handle the presence or absence of placeholder text. `Backbone.Event`’s [`trigger` method](http://backbonejs.org/docs/backbone.html#section-22) allows us to pass the name of a method we want called on an object.

This is great, but we don’t currently have access to the `@listView` in the item view; it’s undefined. Thankfully, the [process of creating a Backbone view](http://backbonejs.org/docs/backbone.html#section-128) takes an options hash, so we can pass in the list view when we instantiate the item view. Let’s do that now:

<script src="https://gist.github.com/thenickcox/de404094708d2f7bea80.js">
</script>

Great. You can see (line 5 in the gist above) that in the instantiation of the item view, we’re setting the value of the `listView` attribute to `this`, which in this case is the instance of the list view. Now we need to tell our list view how to respond to the `handlePlaceholder` trigger we’ve specified in the item view. We can do that with `Backbone.Event`’s `on` method, which takes three arguments: the name of the trigger, the method we’re going to call in response, and the value of the `this` keyword inside that method. Let’s place this in our `initialize` method for the list view:

<script src="https://gist.github.com/thenickcox/07dc6b4796d40bd7f915.js">
</script>

Here, we’re saying `@on` (`this.on`, `this` being the list view) the `handlePlaceholder` trigger, call `this.setPlaceholder`, and make the list view the value of `this` inside that method. So let’s write that method. Spoiler alert! We already wrote most of it in our item view above:

<script src="https://gist.github.com/thenickcox/89f74b8eed2c601aae96.js">
</script>

Fantastic. Now let’s go back to our console and make that album featured again (`Album.where(title: 'A Love Supreme').first.update_attribute(:featured, true)`), refresh the page, and click the `unfeature` link. We should then see the placeholder text. Awesome! Now let’s refresh and see if it still works…

Bummer. Nothing shows up at all. There are no featured albums and no placeholder text.

That’s a relatively easy problem to solve now that we’ve decoupled our list and item views. Let’s call `setPlaceholder` in the initialize method, so it’ll get called when the list view gets instantiated, which happens just after the page load, where we’re currently seeing it blank:

<script src="https://gist.github.com/thenickcox/89ca60b59e22473290b7.js">
</script>

Now make the album featured again and refresh. Well, that’s close. Now we have both the featured album and the placeholder text. So let’s add some logic to our `setPlaceholder` method. We’re going to check if there are any featured albums (known by the number of `li` elements in our list view’s `el`), and set the placeholder text if that’s the case. Otherwise, let’s remove the placeholder text altogether.

<script src="https://gist.github.com/thenickcox/9914b98d681a71d5a57a.js">
</script>

Well, now we’re back to the same problem: on page load, there isn’t any placeholder text. Why is that? Well, let’s think about when the list view will know whether or not there are any featured albums. It’s really not until the collection gets the response back from the server, right? Currently, this method is being called before the collection returns. Thus, there are no featured albums to show. Luckily, the `fetch()` method we’re calling on the collection takes a `success()` callback. This means we can tell it what to do when it returns.

<script src="https://gist.github.com/thenickcox/ab3ed7238dc3bf90294f.js">
</script>

Here, we’re checking the length of the `featured()` method in the collection instead of checking the DOM. Now refresh without resetting our album to featured. Great. We have the placeholder text on page load, but if we do set the model back to featured and refresh, we get both the model and the placeholder text again. What if, instead of checking for the number of `li` elements in our `$el` on page load (which will always be empty on page load, because, again, that’s before the collection returns from the server), we checked for the number of featured albums in our collection before it renders?

<script src="https://gist.github.com/thenickcox/294267c978a67037a540.js">
</script>

Now make sure the album is set to featured and refresh. You should see just the featured album in our featured box, and when you click “unfeature”, the placeholder text should appear. If you refresh, you should just see the placeholder text. You’re done with this step!

## Step 10: Refactoring ##

It feels great to have our app feature complete, doesn’t it? We’ve built our app so that it dynamically displays a series of featured albums that you can unfeature with a single click and without a page reload. It’s fun to play with! But at this point, we should resist the temptation to ship it as is. Why? A few reasons.

Firstly, our app has reached 46 lines, including whitespace. If we add much more to it, it will quickly become larger than we can conveniently fit in our head at once. At that point, maintaining our app will become a cognitive burden.

Secondly (and relatedly), while we’re working in one part of the application, it creates a lot of noise to have to look at every part of it at once. If I just want to work with our albums collection, I shouldn’t also have to look at our albums list view which, while related, again adds to the amount of information I have to keep in my head simultaneously.

Thirdly (and arguably most importantly), if we think about testing this app right now, seeing it as one large file encourages us to think of testing the entire app as one large piece of functionality. Granted, we’re not writing tests in this tutorial (if you’d like to see that in a future post, let me know via email or in the comments), but if we were, it would behoove us to think of our app as discrete components that we can develop and test in isolation from one another. That is, after all, the point of this modular architecture. It’s also how we’re writing our Rails application. If we’ve decided that this style of development is beneficial on the server side, it follows that we’d want to apply that to the client, as well.

Okay, we’ve agreed that we want to break out each component into its own file. Where to start? A good starting point is to take another look at our `application.js` file:

<script src="https://gist.github.com/thenickcox/7705f4f62ec41dfe4d73.js">
</script>

Following the load order, the `application.js` file itself is going to be loaded before any of the components, which will attach our `App` object to the global namespace (which, in the browser is `window`) and give us `Models`, `Collections`, and `Views` namespaces for those components. It will then include those directories, and finally, our `app` file, where we’ll initialize them.

Let’s look at refactoring in the order we’ve laid out in our `application.js` file. The first `tree`, so to speak, is our `templates` directory. This will be the hardest part because it actually requires writing new code. After this, it’s smooth, copy-paste sailing.

We’re going to rewrite the Underscore template from our item view in haml.js. The syntax will look familiar, since the rest of our markup is written in haml. Let’s create a new file called `app/assets/javascripts/templates/album_template.jst.hamljs`. Recall that our template is simply a link to the album’s show page, its title, and a button with the class of `unfeature`. This will make for a very concise haml.js template:

<script src="https://gist.github.com/thenickcox/4694a8c14700a4b8d224.js">
</script>

That should look familiar, because it is, in fact, valid haml.

There are a couple more things we have to do to get this rendered in place of our Underscore template, the first of which is something of a “gotcha.” The `backbone_on_rails` gem gives us a lot of great things, but by default, it puts the JS templates directory under `app/assets`, whereas all the other components, such as models, are placed in `app/assets/javascripts`. Our template actually won’t work with our app configuration without moving that `templates` directory into `app/assets/javascripts/templates`, along with the other component directories. So we have to make a one character change to the change in our `application.js` file to require the correct path to the `templates` directory. Change this:

<script src="https://gist.github.com/thenickcox/8516accb3ae894cad527.js">
</script>

to this:

<script src="https://gist.github.com/thenickcox/c44bb16cfdd5d3e202d3.js">
</script>

By deleting that one period, we’re telling our `application.js` file to look for the templates directory not one directory up (which would be `app/assets`), but in the current directory. Great. Last step: tell Backbone where to look for the template with the JST object in our item view, still in `app.js`:

<script src="https://gist.github.com/thenickcox/ad77b8be7e37dbfb17cb.js">
</script>

If you’re wondering about that JST object, it represents the interface to the [JST ("JavaScript Templates") engine](https://code.google.com/p/trimpath/wiki/JavaScriptTemplates) that translates a templates into executable JavaScript.

Great. Now if we refresh, we should see no changes to the app’s functionality. That’s key with this refactoring. Since we don’t have tests, we’ll have to manually test that the app is still working in between each step. Notice how much cleaner that makes the template.

Since we’re working with the item view, let’s break that into its own file, too. We’re basically going to cut the whole thing and paste it into its own file: `app/assets/javascripts/views/album_item_view.js.coffee`. Paste it in:

<script src="https://gist.github.com/thenickcox/a33efa441dfaae694369.js">
</script>

Now refresh. Everything should work, and the JS console should not have any errors.

Okay! We’re well on our way. Now let’s start at the top of our `app.js.coffee` file with the model. Here’s now what our `app/assets/javascripts/models/album.js.coffee` file should look like:

<script src="https://gist.github.com/thenickcox/0330cfb0d64852f1f15f.js">
</script>

We’re on a roll. Why stop here? Same thing with the collection:

<script src="https://gist.github.com/thenickcox/dcab6393c6f9c8e0622f.js">
</script>

Still no errors in the console? Great. One more:

<script src="https://gist.github.com/thenickcox/a08b567e32aa119c8618.js">
</script>

Great. Now all that should remain in our `app/assets/javscripts/app.js.coffee` file is the code we wrote to bootstrap our application on `document.ready`:

<script src="https://gist.github.com/thenickcox/3a71c9d08cdacf31de45.js">
</script>

How does that feel? If you’re like me, and while you were cutting and pasting the code, you kept your text editor windows open, you might feel significantly lighter now that you’re looking at your app in discrete files. Now we know that if we want to work in one part of the application, we don’t have to be distracted by the noise of several other components.

## Step 11: Show a spinner while records are fetched ##

The great thing I notice about refactoring my code is that it makes me feel like improving the interface or experience of the application more palpable. I find when I’m not trying so hard just to get something to “work,” and I’ve written code I consider production quality, I can worry about other details.

In this case, it’s pretty jarring that when you load the page, it pauses for a second, and the featured album abruptly appears. I’d love to show a loading spinner while the records are being retrieved. Luckily, that isn’t difficult.

We’ll place the spinner in the `.featured-albums.panel` markup, and on the `success` callback from the `fetch()` method on the collection, we’ll hide it. Let’s start by choosing a great loading image.

After a small search, I found these [excellent and simple pure CSS spinners](http://projects.lukehaas.me/css-loaders/) on GitHub. Let’s use one of those. Let’s also add a header. So now our featured albums panel should look like this:

<script src="https://gist.github.com/thenickcox/22a9b3d7335d7892d864.js">
</script>

You can write your own CSS or have a look at the [diff in GitHub](https://github.com/thenickcox/backbone-rails-demo/commit/dad46829485039d18755ea235586e4ad4972be1a#diff-dcf83eacde1973134b408ff229fe6620R21) at the CSS I wrote for the loading image; designing a great loading image isn’t really in the scope of this article.

On the document load, we’re going to tell our list view to hide the loading image before it calls `setPlaceholder()`. Let’s take a first stab at it:

<script src="https://gist.github.com/thenickcox/1ffeeb0b33a35411e544.js">
</script>

Now refresh. Well, it works, but that jQuery `hide()` call feels a little out of place. That essentially makes the albums collection manage the DOM in response to its successful request. And if you recall, we’ve been very careful to keep DOM logic out of the collection. That part of the DOM is the domain of the list view. Let’s refactor this by placing the call in there:

<script src="https://gist.github.com/thenickcox/9cdaf1c391261bf7471c.js">
</script>

You may be asking why we’re returning `this` (which is the list view itself). We’re borrowing this technique from our `render()` method in our item view. Because we’ve returned `this`, when we go back to the `success()` callback, we can now chain two method calls off of `App.albumsListView`:

<script src="https://gist.github.com/thenickcox/4fc8e75466bebd39d1a4.js">
</script>

Now this is much more declarative, and nicely removes the duplication that would’ve been necessary had those been two separate method calls. The list view is managing its own state in response to the album collection’s `fetch`; that way, the collection can do what it does best: reflect the state of the models it’s in charge of.

Excellent. Now we have an attractive loading animation. Even though the user only sees it for a split second, it makes the overall experience much less jarring. Let’s commit our code and take look at the summary of what we’ve done.

## Summary ##

We’ve covered a lot of ground in this tutorial, so let’s look at the lay of the land. From a high level, we took the requirements for a new feature and added it to an existing Rails app using Backbone. But we could’ve accomplished the same task with a few dozen lines of chaotic jQuery. Instead, we wrote organized, highly modular front-end JavaScript with the help of the Backbone library, treating our Rails app as a JSON API. We also learned a few key concepts for writing maintainable object oriented JavaScript: information hiding and the single responsibility principle. We even wrote an implementation of the observer pattern!

Our overall goal was to get a broad introduction to the key concepts of the Backbone.js library: models, collections, views, and templates, and the ways that they communicate via events and callbacks. We also looked deep into the guts of certain parts of the Backbone library. I hope you’ve found, as I have, that the more you dig around under the hood, the less scary it becomes. After all, as of this writing, the entire library weighs in at just 1,736 lines of code, including whitespace. The [entire annotated source](http://backbonejs.org/docs/backbone.html) can be read through in an evening, should you be so inclined.

There are a few topics we didn’t cover. Importantly, we didn’t even look at Backbone’s [router](http://backbonejs.org/#Router) because it wasn’t in our feature requirements to, say, make a URL for a certain UI state linkable. If you’re looking to dive more deeply into Backbone, I would recommend digging into the router. And now that you’re familiar with the rest of Backbone, the concepts introduced in the router should follow naturally.

If you’re a dedicated Rubyist, you might have also squirmed a bit at the distinct lack of automated tests of any kind. My goal in the beginning was to write the introduction to Backbone tutorial that I would’ve liked to have read, and I didn’t want to get bogged down in testing, which would be a tutorial about this length in itself. I’d encourage you to look into the few excellent Ruby gems for adding support for JavaScript testing to Rails, particularly [Konacha](https://github.com/jfirebaugh/konacha). If you’d benefit from seeing those implemented, I’d welcome you to [file an issue](https://github.com/thenickcox/backbone-rails-demo/issues) for this repo on GitHub.

If you’ve gotten this far, thanks for reading, and let me know if there are any Backbone topics you’d like to see in a future tutorial!
