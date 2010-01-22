---
layout: post
title: DRYing up cucumber steps with should_or_not
categories:
  - eden
  - cucumber
---
When using [cucumber](http://cukes.info), I often find myself having to define both the positive and negative consequences of an action. For example, if there were certain cases where the edit button on a page would appear, I would need to test both when the button will appear and when it won't.

{% highlight ruby %}
Then /^I should see the edit button$/ do
  response.should have_tag('input[type=submit][value=Edit]')
end

Then /^I should not see the edit button$/ do
  response.should_not have_tag('input[type=submit][value=Edit]')
end
{% endhighlight %}

There's a lot of duplication going on here -- by [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)ing this up I can save myself trouble later if these steps need to change. My first attempt to reduce duplication started with the realisation that the expectation (should or should_not) is in the step definition itself:

{% highlight ruby %}
Then /^I (should|should not) see the edit button$/ do |should_or_not|
  response.send should_or_not.gsub(' ', '_') have_tag('input[type=submit][value=Edit]')
end
{% endhighlight %}

By replacing the space in "should not" with an underscore, I have the name of the method I need to call on the response. I can simply send this with the have_tag as an argument. This still isn't ideal -- it's arguably harder to read than the original code and the gsub isn't pretty or DRY itself -- but it reduces the amount of code and the minimises risk of change.

The next iteration of should_or_not happened more or less by accident: I once forgot to include the gsub in a step definition. Ruby gave me an undefined method error for 'should not' rather than a syntax error. So does this mean you can define methods with spaces in the name?

{% highlight ruby %}
class Foo
  define_method 'foo bar' do
    "baz"
  end
end
puts Foo.new.send('foo bar')
#=> "baz"
{% endhighlight %}

Yes! Rather surprisingly this is valid. So all I need to do is add a <code>'should not'</code> method in the same context as <code>should_not</code>:

{% highlight ruby %}
#env.rb
module Kernel
  define_method 'should not' do |*args|
    should_not(*args)
  end
end

#step definition
Then /^I (should|should not) see the edit button$/ do |should_or_not|
  response.send should_or_not, have_tag('input[type=submit][value=Edit]')
end

{% endhighlight %}

This works and the step definition is cleaner, but... <code>Kernel</code>? It's not something I want to be adding methods to unless I really have to, and it hardly follows the [principle of least surprise](http://en.wikipedia.org/wiki/Principle_of_least_surprise). I don't think "but it means I don't have to use gsub in a step definition!" really cuts it as an excuse. 

Luckily, cucumber offers a solution that allows me to keep my clean step definition without opening core classes -- [step argument transforms](http://wiki.github.com/aslakhellesoy/cucumber/step-argument-transforms).

{% highlight ruby %}
Transform /^(should|should not)$/ do |should_or_not|
  should_or_not.gsub(' ', '_').to_sym
end

Then /^I (should|should not) see the edit button$/ do |should_or_not|
  response.send should_or_not, have_tag('input[type=submit][value=Edit]')
end
{% endhighlight %}

There we have it -- the same step definition created in a cleaner way.
