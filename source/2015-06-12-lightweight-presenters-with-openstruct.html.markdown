---
title: Lightweight Presenters With OpenStruct
date: 2015-06-12 02:41 UTC
tags: rails openstruct
layout: post
---

I was pairing with [a coworker](http://github.com/sandyw) the other day, and we ran into an issue that we have started to encounter more and more since we started to break up our monolithic Rails app into microservices. The problem is complex and requires some background, but the solution, I think, is elegant and simple.

[READMORE]

There are a lot of moving parts here, so let me describe the background a bit. In essence, our old Rails monolith mounts a series of Rails engines at different URL endpoints. These engines communicate with the main app via a series of service objects that grab data from our database (that the main app has access to) and return that data to the engines. The goal of this exchange is that the microservices (Rails engines) can be developed in isolation and don't need knowledge of the main app database; instead, in cases such as testing, they can rely on a mocked out service object, the actual version of which is implemented by our main Rails app. This is in case, for example, we replace our main rails app; at that point, the services can communicate with another data source, such as a JSON API, without our entire engine needing to change. You might recognize this as the [façade pattern](https://en.wikipedia.org/wiki/Facade_pattern).

With me so far? Great. When a class (consumer) uses that service object, it will probably end up using that data in a view. In our case, we needed data from two different service objects in the same view, but we wanted to try to follow [Sandi Metz’s rule of only instantiating one object in a controller](https://robots.thoughtbot.com/sandi-metz-rules-for-developers) because we’ve generally found that to produce [simpler](http://www.infoq.com/presentations/Simple-Made-Easy) code.

Our first attempt at this was to wrap the service class’s data (returned as a hash) in an `OpenStruct` that we could use in the view by calling methods on it rather than accessing hash properties. First, our two services:

<script src="https://gist.github.com/thenickcox/1caf496d84a090bf420e.js">
</script>

<script src="https://gist.github.com/thenickcox/a7ea1c6d6aec789bf095.js">
</script>

Now our controller:

<script src="https://gist.github.com/thenickcox/e3ad77ef9139e83182cb.js">
</script>

You can see that the controller effectively gloms the returned data from both services, throws it in an `OpenStruct`, and pushes it out to the view, so that the view can call things like `@person.name` and `@person.mobile_phone` without worrying about their implementation, and we didn’t have the more unsightly `@person[:name]` that we would have had we just returned a large merged hash. This is a benefit because, for example, the object that contained the attributes could be changed and uphold the same API contract, whereas only a hash (practically speaking) can uphold the API contract of a hash.

This was great in theory, but we had decided that we wanted to write the view with the simplest possible code: little (if any) logic, easy-to-read method names, etc. So we wrote the view relying on the `@person` object having a `full_name` method rather than interpolating the `first_name` and `last_name` properties. And the presenter, having inherited from `OpenStruct`, happily swallowed the method it didn't know about and returned `nil`; the view didn't throw an error. This silent failure took us a few minutes to track down when we ran our tests until we remembered that we had inherited from `OpenStruct`.

We then decided to create an object we affectionately named `WhinyOpenStruct`, which inherited from `OpenStruct` and only contained a single method: an implementation of `method_missing` that, when called (as in the case of `full_name`), would raise an error that told the developer which attribute it was missing rather than silently returning nil:

<script src="https://gist.github.com/thenickcox/9fe0cde533303d2a9e8e.js">
</script>

We have [this StackOverflow](http://stackoverflow.com/a/16905766/931934) answer to thank for the implementation.

Our next step was to pass the hash returned from our service object into the creation of our `WhinyOpenStruct`. Our presenter then inherited from `SimpleDelegator`, delegating methods it didn't recognize to our `WhinyOpenStruct`. (If you're not sure how `SimpleDelegator` works, I don't blame you; I hadn't used it before this pairing session, and my pair explained it to me. Effectively, you instantiate a class that inherits from `SimpleDelegator` (our presenter, in this case), and the object will delegate any methods it doesn't implement to the class passed into the constructor (a.k.a., `initialize`).)

Using this approach, we accomplished all of our goals: our view was able to call methods on an object rather than look up values in a hash, our presenter was able to remain elegant, and our presenter object would raise exceptions when asked to respond to attributes it didn't implement. All in a good day's work!
