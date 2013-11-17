---
title: Refactoring Cucumber
date: 2012-12-17 03:56 UTC
tags:
layout: post
---

At work, we’ve been moving away from Cucumber tests for the past number of months, in favor of the more robust Capybara and rspec paring. Given that I’m a relative novice in Rails test-driven development, the supposed brittleness of Cucumber tests wasn’t immediately clear to me, so I did some digging, and came up with a [few](http://aslakhellesoy.com/post/11055981222/the-training-wheels-came-off) [posts](http://benmabey.com/2008/05/19/imperative-vs-declarative-scenarios-in-user-stories.html) that talk about the pitfalls of the “imperative” style of step writing for user stories. READMORE A solid example of this is Ben Mabey’s ancient, but still applicable post, [“Imperative vs. Declarative Scenarios in User Stories”](http://benmabey.com/2008/05/19/imperative-vs-declarative-scenarios-in-user-stories.html). Mabey’s assertion is that it is too easy to rely too heavily on the built-in Cucumber web steps. For example:

<pre><code>
When I fill in Name with 'Alligator'
And select Phylum as 'Chordata'
And fill in Animal Class with 'Sauropsida'
And fill in Order with 'Crocodilia'
And fill in Family with 'Alligatoridae'
And fill in Genus with 'Alligator'
And check Lay Eggs
And click the Create button
</code></pre>

Here, Maybe suggests, the “story” part of the user story tends to get lost and mired in the technical details. I was heavily influenced by this post, and decided to do a little Cucumber refactoring at work to try out Mabey’s declarative style, which he demonstrates thusly:

<pre><code>
Story: Animal Submission
  As a Zoologist
  I want to add a new animal to the site
  So that I can share my animal knowledge with the community

  Scenario: successful submission
      Given I'm on the animal creation page
      When I add a new animal
      Then I should see the page for my newly created animal
      And the notice 'Thank you for your animal submission!'
</code></pre>

As Mabey notes, it has more of the [token for conversation](http://www.jbrains.ca/permalink/user-stories-a-ticket-for-a-conversation) feel to it, and it’s immediately clear that it is more likely to meet the user’s need, since, in this case, the user cares more about completing their goals than how those goals are handled on the back end.

## My Attempt

I took Mabey’s advice to our own codebase, and I extracted a Cucumber test that I had written a few months back. Now, I won’t start to bad mouth my code and talk endlessly about how terrible a person I am for having written it this way, but I’ll give you a taste of how poorly this test spoke to the user’s need. I present to you this feature test for our site’s feedback form:

<pre><code>
#features/feedback.feature
  Feature: Feedback
  Scenario: Submitting feedback
Given I go to the feedback page
And I fill out the feedback form and submit it
Then I should see a success message
Then I should see "Email can't be blank"
When I fill in "Your email" with "nick@navigatingcancer.com"
And I fill in "Summary" with "This site rules"
And I fill in "What you observed" with "This site rules"
And I press "Send"
Then the flash notice should be "Thank you for your feedback! |
  We try to respond to questions and requests within one business day."
And ruby FeedbackForm.last.email.should == 'nick@navigatingcancer.com'
And ruby FeedbackForm.last.content.should match "This site rules"
</code></pre>

Yep, you read that right. That’s a Ruby function inside of a Cucumber test. I wonder what user will be thinking of their form submission in terms of a Ruby method chain? Now here’s my improved version, written with Turnip and Capybara.

<pre><code>
#spec/acceptance/forms/feedback.feature
  Feature: Feedback form

	As a user of the site
  	I want to be able to submit feedback about the site
  	So that I can report if something breaks

	@feedback
 	Scenario: Submitting feedback (successful)
    Given I go to the feedback page
    And I fill out the feedback form and submit it
    Then I should see a success message
</code></pre>

<pre><code>
#spec/acceptance/forms/feedback_steps.rb
  require ‘spec_helper’

  steps_for :feedback do
    step 'I go to the feedback page' do
      page.visit '/feedback'
    end

    step 'I fill out the feedback form and submit it' do
      page.click_button 'Send'
      page.should have_content "Email can't be blank"
      page.fill_in 'Your email', with: 'joe@navigatingcancer.com'
      page.fill_in 'Summary', with: 'Just saying'
      page.fill_in 'What you observed', with: 'I am awesome'
      page.click_button 'Send'
    end

    step 'I should see a success message' do
      page.should have_content "Thank you for your feedback! We try to respond to questions and requests within one business day."
      FeedbackForm.last.email.should == 'joe@navigatingcancer.com'
      FeedbackForm.last.content.should match 'I am awesome'
    end
  end
</code></pre>

See how much more nicely that reads? Now the user story is really coherent. They go to the form and submit it, and they should get a success message. That’s truly how simple the step should be. Notice that I still handled the validation in Capybara. But the it’s not the user’s job to think about what should happen if they don’t submit the form correctly, it’s mine. Thus, I should handle that in the test logic that doesn’t involve the user story.

## Onward##

As a front-end developer coming to the TDD Rails world, Cucumber was helpful to me because it helped me to think about things from the browser’s perspective. The gherkin syntax, when used well, really does allow for expressive steps and user stories that invite collaboration with non-developers. But that syntax paired with Capybara’s more robust step handling make for a much clearer and well-executed test.

Notably, the reason I went back to this test was that we had changed the UI for the form. Our previous form submitted an unfuddle ticket, but we have recently switched our ticketing system to Assembla. Also, we added a few fields and renamed others. In this case, the Cucumber test (previous to the one I have posted) would have failed because it wouldn’t have been able to find the form labels it was looking for. But the user story remained the same. All the user wants to do is submit a feedback form. So rather than changing several tests to reflect this, why not only change the ones that are concerned on the back end? They’re easier to maintain, and as great as Cucumber was, I find that I prefer the Capybara-rspec approach when it comes to looking for content on the page. I’m looking forward to exploring this technique across our codebase during refactoring, and in the future.


