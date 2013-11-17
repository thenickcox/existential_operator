---
title: Method Aliases in Ruby
date: 2012-12-18
tags:
layout: post
---

I often find myself wondering why so many of the Ruby native string and array methods have aliases. For example, with string methods, we have `next` and `succ`, which are equivalent; they both return the next alphanumeric character. Like so:


<pre><code class='bash'>
1.9.3p194 :024 > string = "a"
 => "a"
1.9.3p194 :025 > string.next
 => "b"
1.9.3p194 :026 > string
 => "a"
1.9.3p194 :027 > string.succ
 => "b"
</code></pre>

So why do we need both? READMORE Here is another example. length and size are also synonymous. Both return the character length of a string.

<pre><code class='bash'>
1.9.3p194 :028 > a = "supercalifragilisticexpealidocious"
 => "supercalifragilisticexpealidocious"
1.9.3p194 :029 > a.size
 => 34
1.9.3p194 :030 > a.length
 => 34
</code></pre>

This is perplexing to me. As much as I love Ruby, I often think that my mind philosophically prefers Python’s approach. The following is a direct quote from [‘The Zen of Python’](http://www.python.org/dev/peps/pep-0020/):

> There should be one—and preferably only one—obvious way to do anything.

I don’t necessarily have an answer (other than the likelihood that one or the other will be more familiar to programmers coming from other languages), but here’s a thought. We’re not working in Python, so let’s embrace the fact that Ruby has two ways of doing things sometimes.

If we think intuitively, it seems to me that we would want to find the ‘size’ of an alphanumeric string like, for instance, a base 64 token. That, to me, makes more intuitive sense than to get the ‘length’ of it. `length`, to me, seems to be the method we would want to use to find the length of a word, where the word ‘size’ (and method of the same name) don’t make as much lexical sense.

For example:

<pre><code class='bash'>
1.9.3p194 :018 > token = SecureRandom.urlsafe_base64
 => "OFiTxLeCuosWXiLe9nVLeQ"
1.9.3p194 :019 > token.size
 => 22
</code></pre>

Compare that to this:

<pre><code class='bash'>
1.9.3p194 :022 > sentence = "An incredibly long sentence about whosits and whatsits"
 => "An incredibly long sentence about whosits and whatsits"
1.9.3p194 :023 > sentence.length
 => 54
</code></pre>

Doesn’t that just make more sense? To me, it seems more expressive.

Just to reiterate, whereas a developer coming from Python might be tempted to use one or the other method uniformly throughout a project, I like to think that Ruby’s features lend themselves well to expressiveness, and I would prefer using one method or the other in the context it makes sense. After all, we work in a language with this syntax:

<pre><code class='ruby'>
3.times do 'Hello World!'
</code></pre>

This is as close to spoken English as I’ve ever seen programming syntax. So let’s write Ruby like we would speak, and use methods where they make sense contextually, even if they do the same thing. As a result, our code will be eaiser to read, and therefore maintain. That’s why we have the canonical saying:

> ‘Programs must be written for people to read, and only incidentally for machines to execute.’
> - H. Abelson and G. Sussman “The Structure and Interpretation of Computer Programs”

That sounds good, but let’s make this a little bit more complex. Look what happens when we make our string an array. size and length still apply, but to count the number of items in an array we also have, you guessed it, count. Check this out:

<pre><code class='ruby'>
1.9.3p194 :017 > sentence = "An incredibly long sentence about whosits and whatsits"
 => "An incredibly long sentence about whosits and whatsits"
1.9.3p194 :018 > a = sentence.split(' ')
 => ["An", "incredibly", "long", "sentence", "about", "whosits", "and", "whatsits"]
1.9.3p194 :019 > a.size
 => 8
1.9.3p194 :020 > a.length
 => 8
1.9.3p194 :021 > a.count
 => 8
</code></pre>

So when should we use which? According to the Ruby docs, size and length are truly the same, but count can take a block, and will return the number array items that give a true value. So let’s look at our previous example with this in mind:

<pre><code class='ruby'>
1.9.3p194 :027 > a
 => ["An", "incredibly", "long", "sentence", "about", "whosits", "and", "whatsits"] 
1.9.3p194 :028 > a.count
 => 8 
1.9.3p194 :029 > a.count('long')
 => 1 
1.9.3p194 :030 > a.count('hello')
 => 0 
</code></pre>

So while count can return the number of array elements, it also has this added feature of counting the number of passed in items the array elements match.

Thus, the only rule I can think of when working with arrays is to choose size or length and work with it throughout a project to return the number of array items, but only use count when you need to pass in a block so that there is no confusion about why you used differerent methods in different places to do the same job. This will lend to more cohesion and more expressiveness.

## Gotchas

There are, of course, some gotchas to this idea. When I first started writing Ruby, I was so in love with the English-like syntax that, for a time, I decided I wanted to use words instead of symbols for conditional operators. Thus, when I was a beginner, I might’ve at one point been prone to write:

<pre><code class='ruby'>
def greeting
  if user.logged_in? and not(user.nil?)
    "Welcome back, #{user.name}"
  end
end
</code></pre>

That was a contrived example, for sure, and an exaggeration, but I wanted to get to this point: While researching the logical operators, I found a [case](http://blog.internautdesign.com/2007/5/30/ruby-operator-precedence-gotcha-logical-and) where and and && returned different values:

<pre><code class='ruby'>
1.9.3p194 :013 > result = true and false; result
 => true
1.9.3p194 :014 > result = true && false; result
 => false
</code></pre>

That scared me enough to make me want to use symbolic logical operators all the time and sometimes parenthesis just to communicate intent. I now do this even when not strictly necessary:

           def viewable_by?(current_user)
       current_user.system_admin? || ( current_user.logged_in? && current_user.owner? )
     end

In this case, the parenthesis weren’t absolutely necessary, as can be seen when we boil this example down:

<pre><code class='ruby'>
1.9.3p194 :024 > false || true && true
 => true
</code></pre>

But I use parenthesis anyway. Doing so doesn’t sacrifice the aesthetics of the code (in my opinion, anyway), and that way, my code intentions become immediately more clear. And isn’t that the goal?


