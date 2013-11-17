---
title: A Public Apology to Nicole Sullivan
date: 2013-11-16 22:54 UTC
tags:
layout: post
---

I'd like to offer [Nicole Sullivan](http://stubbornella.org) a public apology for dismissing her work. But first, a bit of back story.

READMORE

When I started my current job, I inherited the front-end of a four year old Rails app built exclusively by the engineering team. That is to say, the company had never had a formal design team. In particular, they had never had a front-end developer responsible for the content and organization of the CSS.

Consequently, the application's basic skeletal structure was as follows:

          <body class='[rails-controller-action] [dynamically-applied]'>
      <div id='content'>
        <div id='header' class='branded-header'>
        </div>
        <div id='page'>
          <div id='secondary' class='sidebar'>
          </div>
          <div id='primary' class='main'>
          </div>
        </div><!-- /page -->
      </div><!-- /content -->
      <div id='footer'>
      </div>
    </body>

You read that right. And the CSS was all written such that not only were both the id and class attributes used as styling hooks (that's right, both), the project was using Sass. And if there was one thing the developers knew about Sass, it was that you could nest rules, like so:

<pre>
  <code class='scss'>
#primary
  div.green-box.shoutout
    color: $green
    h1.black-header
       color: $black
  </code>
</pre>

The point here, which is likely obvious by now, was that the stylesheet was the absolute prototype of everything not to do in a stylesheet. It was not semantic (`div.green-box`). It was overly specific (in the above case, the selector in the compiled CSS was: `#primary div.greenbox.shoutout h1.black-header`). The markup lent itself to poor CSS, and the CSS required more and more markup classes to override the already overly specific styles, and so on. It was a maintenance nightmare.

Further, it was large. We recently reached IE's [CSS selector limit](http://blogs.msdn.com/b/ieinternals/archive/2011/05/14/10164546.aspx). And the compiled CSS was upwards of 20,000 lines.

A few weeks ago, the decision was made from our managerial team to do a top to bottom redesign of the entire application. I was joyous, though it was a daunting task. When I began to plan the markup and CSS changes, I knew that I had to implement something with a solid foundation of design patterns that wouldn't create any overhead. A modular framework that would allow the developers to throw elements on a page, add some classes to them, and only begin to write CSS if absolutely necessary.

Over the past few years, I had repeatedly stumbled upon both Nicole Sullivan's [Object Oriented CSS](http://oocss.org/) and Jonathan Snook's [Scalable Modular Architecture for CSS](http://smacss.com/), both of which I met with a sigh and an eye roll. I reasoned that, as I had been writing CSS for seven years, there were few new things I could practically learn, with the possible exception of the exact order of vendor prefixes on obscure and experimental CSS3 features that I never used. So, I decided, there was no reasons to even read further.

Back at work, I had made the firm decision that, though the developers weren't going to be happy about writing a smattering of classes in their markup, the alternative was much less preferable. I realized that I could take the guesswork out of the design process for my colleagues by pre-designing a set of standard user interface elements, implemented through reusable CSS patterns and the judicious use of CSS classes. I started digging into the source of modular frameworks like [Twitter Bootstrap](http://getbootstrap.com/2.3.2/) and the [Foundation](http://foundation.zurb.com/) framework, and the idea I had believed I generated was confirmed. I knew that ideas like this were going to be central to the way I wrote our markup and CSS going forward.

This morning, while enjoying my [Instapaper](http://instapaper.com) queue alongside a cup of [excellent coffee](http://foundation.zurb.com/), I came across [MailChimp's UX patterns library](http://ux.mailchimp.com), which perfectly embodied everything I was looking for: a complete style guide that outlined just about every element included in their application that gave the interface a unified, consistent feel across platforms and devices.

Browsing through, I noticed an acknowledgement of the work of Nicole Sullivan, and begrudgingly followed the link. After only a few minutes of reading her work on GitHub, I came to the humbling realization that I had been influenced by her work all along. That what she had been advocating for a number of years was exactly the idea that I had so presumptuously thought I had originated on my own. And that the approach she outlined was innovative, important, and helped progress the craft of writing CSS.

So why, then, had I dismissed it?

## Pride ##

As I mentioned before, I had been writing CSS for seven years. I had reached a point where any user interface problem I came up against only helped to confirm my suspicion that, if I could imagine it, I could code it. I had concluded that my CSS was beyond reproach and that there was nothing more worth learning. I was wrong.

## Misunderstanding ##

Part of what I disliked about Nicole's approach was its title. How, I asked incredulously, could CSS be object oriented? As Sandi Metz says in her excellent book [Practical Object-Oriented Design in Ruby](http://cloud.nickcox.me/SWTc), the heart of object orientation is the sending and receiving of messages. Because CSS doesn't have methods, it could never be truly object oriented, I reasoned. I knew about Sass functions, and even the `@import` directive that theoretically mimicked classical inheritance (if you squinted your eyes a little), but I held firmly to the belief that CSS, by definition, was not a programming language (it had no concept of data types, a necessary prerequisite as I saw it) and could not, therefore, ever be object oriented. In short, _I had openly dismissed her work as inaccurate without ever having familiarized myself with it, simply because of what it was called._ I was wrong.

## Sexism

That's a tough headline to write beneath, but I have to acknowledge that a lot of my bias was likely due to sexism. After my realization that I had naïvely thought I had come up with an idea that Nicole championed nearly four years ago, I had wondered if my response to her evangelism would have been different if it had been popularized by Paul Irish or Dan Cederholm. Here is where, if I was trying to save face at all, I would claim that I was equally dismissive of Jonathan Snook's work. I would suggest that I couldn't possibly be guilty of sexism, because I was prejudiced against their work equally. But as a white, upper middle-class, cisgendered, heterosexual male, I've learned that if any thought I have smacks of sexism (or heterosexism, or racism, etc.), it probably is. As far as missing Jonathan's work, it's still a bias, just another kind. I was wrong.

## I'm sorry, Nicole. And thank you.

I'd like to conclude with a personal note to Nicole. Nicole, if you're reading this (you see? I'm already assuming that my work is important enough for her to address directly), I'm sorry that I dismissed your pioneering work, for a number of reasons, not the least of which was because you're a woman. I was wrong. As a member of a community in which a great deal of violence has been wrought against women lately, I already know (though I could never fully understand from the harrowing firsthand experience that you and your female colleagues undoubtedly experience frequently) how difficult it must be to be a woman in tech and to try to be taken seriously.

I'd like to acknowledge openly and publicly that I've been greatly influenced by your work, and I find it innovative, creative, and extremely helpful for the use case I'm currently working on. You've helped solve a large problem for my codebase at work, and the longevity of our application, and for that, I'm grateful. Thank you.

In the future, I look forward to being continually humbled by both your work and the work of your female contemporaries. I just hope that, marked by this experience, I can acknowledge it sooner.

Again: I was wrong. I'm sorry. And thank you.

## Epilogue

If you read this and immediately asked "What was the point of admitting this publicly?" and/or "This isn't fair; it's not Nicole's job to bear the burden of your feelings of guilt," I would consider both of those to be fair criticisms. But if I were to just acknowledge this to myself and go on with my day, I wouldn't really feel the extreme discomfort of the vulnerability that writing and posting this piece actually took.

And the choice to avoid that discomfort is the very _definition_ of privilege; women, people of color, and members of other marginalized communities don't get to choose to not be part of the conversation on a day-to-day basis.

So my hope is less that Nicole herself reads this and, if she does, absolves me of this transgression (though I would welcome that if it were possible), it's to further the dialogue about the problem of sexism in tech, and to have to face the realization that, though I don't consider myself sexist, racist, homophobic, or otherwise biased, I absolutely can and do make mistakes that contribute to the systemic oppression of these people, well intentioned though I may be.

And I hope that this encourages other people of privilege to confront that privilege, recognize when they oppress the underprivileged, and apologize in a way that both helps them to learn from the error, to experience the discomfort of remorse, and, with any luck, to make them that much less likely to commit that same act of oppression in the future.

