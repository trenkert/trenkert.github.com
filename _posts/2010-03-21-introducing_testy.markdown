---
layout: post
title: "Introducing Testy"
categories:
  - eden
  - apprenticeship
  - rfc2616
  - testy
---
Both [Richard](http://edendevelopment.co.uk/blogs/richard/) and I have been set a rather daunting new task by our mentor [Chris](http://chrismdp.github.com/): to implement, in Ruby, a web server compliant with [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616.html) (<em>sans</em> caching and the Accept header). To successfully complete the task, we must be able to serve static HTML pages to a modern browser, but we get bonus points for other content types and, in Chris' words, "super bonus points" for interfacing with [Rack](http://rack.rubyforge.org/). One catch: we're only allowed to use the standard Ruby libraries to complete this, no gems allowed.

As my implementation must be [test-driven](http://en.wikipedia.org/wiki/Test-driven_development) and I can't use Test::Unit, the first part of the challenge is to build a simple test framework. Luckily, I already have one just lying around! [Aimee](http://edendevelopment.co.uk/blogs/aimee/) and [Spencer](http://edenspencer.github.com/) both had to build test frameworks for their first apprenticeship task and it sounded like an interesting challenge, so in a spare hour or two I created [testy](http://github.com/ohthatjames/testy).

I tried to keep testy simple: it only has an <code>assert</code> method, which checks whether the given expression evaluates to true. I may add more later as I need them. Failed assertions raise an exception that is caught by the TestSuiteRunner. This is an easy way of stopping execution of the test and providing debug information.

The <code>TestRunner</code> takes a directory on initialisation and loads every file in the directory. I wanted to avoid forcing the same file name and class name for my test suites mainly to allow more than one TestSuite per file. This makes it possible to use multiple TestSuites in the way one can use "describe" blocks in RSpec files. To accomplish this, each time TestSuite is subclassed, the suite is registered at the TestSuite class level using the Ruby <code>self.inherited</code> hook. This may come back to bite me later as it's not very flexible, but at least it'll get me started.

And just because I was playing around I added some features that, while not strictly necessary, should make development a bit easier:

* Any exceptions are caught and processed as a failure to stop the whole suite from bombing out.
* Added a <code>setup</code> method to the TestSuite so that shared setup of tests can be extracted.
* Pending methods, just because it was easy after writing the <code>assert</code> method.

The next step is to finish reading RFC 2616 and then decide on how to begin. I imagine I'll be playing around with Ruby sockets for a while, but more importantly I'll need to be thinking about what and how I'm going to test this application. As I'll be outside of my comfort zone of [RSpec](http://rspec.info) and [Cucumber](http://cukes.info/), it may be time to reevaluate how I test my code.

Attending [Corey Haines'](http://www.coreyhaines.com/) excellent Code Retreat at Bletchley Park got me thinking about how I approach TDD. I naturally tend to write more complex code than is required for the functionality I'm building, so I find it quite hard to slow down and really think about what I'm trying to achieve: am I breaking my tests down into small enough units of behaviour? Am I really doing the simplest possible thing to make my tests pass?

I also need to give some thought to how granular my tests will be. Cucumber advocates an outside-in approach to development, but what if [integration tests really are a scam](http://www.infoq.com/presentations/integration-tests-scam) as J.B. Rainsberger [asserts](http://www.jbrains.ca/permalink/239)? How much value does automated testing of larger parts of an application really provide?

And then there's Rack. Is it simpler to ignore Rack because [YAGNI](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it), or will I be shooting myself in the foot if my design just doesn't fit the Rack paradigm easily? I've yet to be really convinced that architectural changes can be driven out from small refactors as new requirements are added: are there some things that just need to be considered up front?

Lots of questions, but no easy answers. I'll be blogging about some of them as I start to flesh out the answers myself. In the meantime, any feedback or thoughts about testy or any of the questions raised here will be gratefully received.
