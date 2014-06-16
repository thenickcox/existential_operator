---
title: How to Choose a New Programming Language
date: 2014-06-14 18:11 UTC
layout: post
---

I had recently felt like my skills in Ruby had reached a plateau. Not at all because I had learned everything; on the contrary, it's just now that I'm gaining an understanding of all that I _don't_ know. Rather, about a month ago, I had felt a stagnation in my skills because all of my programming had been confined to Ruby and JavaScript. And someone recently mentioned the book [Apprenticeship Patterns](http://shop.oreilly.com/product/9780596518387.do), which suggests that to further our career, programmers learn a new language every year.

Because all of my programming was confined to the two languages I was most comfortable with (and the fact that I didn't study computer science in college), there were aspects of programming that I had absolutely no concept of. For example, I had been reading a lot of what Steve Klabnik has been writing on the [Rust language](http://words.steveklabnik.com/a-30-minute-introduction-to-rust), and I was immediately out of my element when he started talking about concepts such as pointers, a concept it seems is immediately familiar to programmers of C++. This is to say nothing of manual memory management or other close-to-the-metal concepts.

In light of all this, I decided to begin the foray into a new language in the hopes of broadening my perspective. [READMORE] I was limited in my scope to what seemed to be gaining traction in my admittedly limited worldview (read: Twitter feed), and there are several that have gotten a lot of attention lately. In particular, Go, Rust, Elixir, Clojure, and Scala rose to the top. I spent a bit of time reading a few articles on each, and these new(isn) languages all looked exciting enough to invest a few months in to find out whether something useful could eventually be built.

In deciding which language to try, I sketched out a number of considerations that I'm including here in hopes of helping anyone else trying to sort through the noise. After all, learning a language is a substantial investment of time and energy. I found these criteria helpful in gauging the likelihood of a favorable ROI on time spent learning a language.

## A Broad Community

Though not necessarily the most important criterion, I find that a community is essential to language learning. And I use the word community broadly, encompassing IRC channels, Stack Overflow activity, discussion boards, and local meet-ups. It was extremely important to me that, if I were going to learn a new language, I wouldn't be learning it in a vacuum.

Does the language you want to learn have a supportive, enthusiastic community built that will help you become un-mired when you reach the inevitable learning curve?

## Availability of Resources

I've worked through enough tutorials, videos, and books to have learned a bit about my learning style. I find that I learn best when I have money on the line. Through a long process of trial and error, I've discovered that I learn the most when I spend at least $30 of my own money on a physical book (and specifically a physical book, not an eBook), and go through it chapter by chapter, physically writing every line of code possible. Thus, there had to be a reputable book available.

I wasn't immediately able to find a huge variety of resources about Elixir or Rust. (The two books on Elixir seem to be in beta, and I've had terrible experiences with pre-release unedited books.) Rust does have Steve Klabnik's excellent ["Rust for Rubyists"](http://www.rustforrubyists.com/) tutorial, but in my case, I've truly learned that it has to be a physical book that costs me money in order to learn.

Does the language you're looking at have the resources that most complement your learning style? Perhaps you're best with video courses like CodeSchool. Can you find what you need to learn in the way that best suits how your brain works?

## Stable API

When I was learning Ember.js in the pre-1.0 time period, I found it really frustrating that the API was constantly changing, and tutorials and articles dated just a few months back would produce confusingly broken code when used with the most recent version of the framework. This repeated frustration led me me to drop learning it altogether until after 1.0 arrived. (It's worth noting that the Ember community has done a great job of improving their documentation, and even had a campaign to edit old Stack Overflow answers to reflect the most current API!)

Rust seems like the most volatile language of the bunch, and I'm aware that, while learning a framework with a changing API is one thing, trying to learn a whole language whose documentation is a moving target would be entering a world of pain. I wasn't ready for that.

What's the current version of the language you're considering? How tolerant are you to APIs that change daily? If you're considering using the language in a production environment, is it ready for primetime? How painful will the upgrade process be if you have to do it frequently? 

## BYOE

I'm a vim addict. Having worked in vim full-time for nearly a year and a half, I view its key bindings as an absolute requirement for any programming work, provided they're available. I also wasn't interested in learning a new IDE in addition to a language. That seemed like too much overhead.

To the best of my knowledge, every language I've mentioned is editor-agnostic, so that question was a wash. But it would have excluded languages like Objective C or C#, had those been in contention. (Of course you _can_ code Obj-C or C# in your own editor, it's just harder to compile, run, or port around than most.)

Can you use your favorite tooling in learning the language you're considering? If not, are you interested in learning a new environment like XCode or Visual Studio to get your work done?

## A Conference

Go, Clojure, and Scala all had conference videos available on [Confreaks](http://confreaks.com). (ElixirConf is [next month](http://elixirconf.com/).) I watched a few to judge the quality of the talks, and to determine whether any talks were abstract enough to be understandable without an understanding of the language syntax. I watched for concepts, such as popular use cases or exciting and innovative qualities of the language. 

I watched a few talks from the Clojure and Go conference, and I had a hard time connecting to the talks (compared to, say, the conference of which I've watched the most videos recently: FutureJS). While it was clear that the speakers were well-informed and alarmingly smart, they didn't seem to communicate their ideas in a way I found accessible.

Has your potential language held a conference? What was the attendance like? If it's available, watch at least the keynote of the conference, which is likely to discuss the development of the language, and how far it has come in relation to where it started. Does it have the trajectory needed to gain a critical mass of colleagues you can work with?

## Wide Support

Say what you want about Java, the ubiquity and maturity of the JVM is undeniable. I had thought about just cutting out the middle man and learning Java, if for no other reason than the sheer number of job descriptions I've seen include it. But from the little Java code I've read, I wasn't attracted to its type system, which offers no inference, and, as a result, requires much more boilerplate code to get going.

But given the benefits of the JVM, I was really interested in a language that would run on it, besides JRuby. This narrowed the field to Clojure and Scala.

How easy is it to find a platform to deploy a program in your language? Can you compile it into an easily distributable package that runs on different platforms? Are PaaS's like [Heroku](http://heroku.com) or [Engine Yard](http://engineyard.com) supporting deployment environments for your language of choice?

## Similar to, yet fundamentally different from what you know

I reasoned that I would be the most productive in a language that offered support for data types or syntaxes that I could grok immediately. It followed that those concepts would produce the least cognitive load, and enable my learning that much more quickly. 

And yet if I were to succeed in my goal of broadening my Ruby abilities, it would have to be a language that would stretch me outside of my comfort zone. I decided that after hearing all the hype, I was was ready to take the plunge and see what functional programming was all about.

Initially, I saw that Scala supported both functional and object oriented programming styles. While this is often touted as a strong point, I was wary of a language with those features. (I know, JavaScript can make the same claim–kind of–and I'm still wary of that aspect of it.) I tend to prefer to use more highly specialized tools designed for a single purpose. 

And yet this seemed like a gateway into functional programming, if I could use both idioms in the same language and grow in both over time. (I am still wondering whether both idioms can or tend to be mixed in the same codebase, or whether that's counterproductive. Time will tell.) Here, Scala emerged as the main contender.

If you're early in your career and considering a language to help grow your skills and introduce you to new concepts, are you likely to find idioms that are at least somewhat similar to what you know? If not, how tolerant are you to having your mind totally bent by a new paradigm? (I still remember reading the [Clojure rationale](http://clojure.org/rationale) and kind of having my mind blown.)

## Hype and popularity

It's true: I'm as susceptible to hype and popularity as anyone. I follow a lot of smart people on Twitter, and it helps to have them filter out choices for me. I'm happy to defer to their expertise.

I would probably say, then, that the language I've heard the most hype about was probably Go, followed by Clojure. (Notably, Ben Orenstein of Thoughtbot recently expressed his obsession with Clojure on an episode of the [Giant Robots podcast](http://podcasts.thoughtbot.com/giantrobots) in a compelling way.)

Let's face it, a lot of things in the programming world aren't al they're hyped up to be. We're extremely susceptible to jumping onto what's in vogue. (Remember when XHTML was the only true way?) That said, you probably follow some really intelligent people on Twitter, and they're thought pioneers for a reason. What are they and others saying about the language you're considering? Let other people do the exploratory work for you and profit from the legwork they're doing to exercise your language of choice. Use that as scaffolding for knowing where to start your learning process.

## Uses

A fair critique of this article would be that the languages I've mentioned are designed for very different purposes, and that the comparisons aren't very fair. While that may be true, I'm not looking to build extremely fault-tolerant, concurrent, or scalable web applications at this point in my career. However, I am looking to gain perspective and a deeper insight into new and different styles of programming. In that regard, any language will help me by challenging me to think differently.

You're hiring the language to do a job for you. Either it's to increase your productivity, to teach you a new style of programming, or to give you the capacity to build concurrency into your application. What was the language you're thinking about designed for? Will it meet the need you're hiring it to fill?

## Just pick one

Acquiring new skills is a skill like any other, that itself requires practice and repetition. Why not just pick a language? If, like me, you're considering several, it might help just to get your learning momentum going. Consider time-boxing your learning. Give yourself one month, for example, and set a goal. Maybe you'll deploy a simple web application. Maybe you'll write a useful command line tool that you can open source. If you end up realizing that the language isn't helping you achieve your goal, then the worst case scenario is that you've practiced learning, and that's a skill we all need to hone.

In the spirit of "Just pick one," I've chosen [Scala](http://www.scala-lang.org/), and I'll be  documenting my progress here. What language have you chosen? [Let me know on Twitter](http://twitter.com/everydaytype) or in the comments.

