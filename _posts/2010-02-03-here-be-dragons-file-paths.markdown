---
layout: post
title: "Here be dragons: Constructing file paths without strings"
categories:
  - eden
  - here_be_dragons
---
Using Ruby provides a lot of "I wonder if we could do that?" moments, little things I play with to satisfy my curiosity rather than any actual purpose. My most recent one came when thinking about how Merb creates file paths from strings:

{% highlight ruby %}
class String
  def /(o)
    File.join(self, o.to_s)
  end
end

"files" / "are" / "here" # => "files/are/here"
{% endhighlight %}

I wonder if we could lose all those extra quotes? Let's start by removing them and see what happens:

{% highlight ruby %}
files/are/here

# undefined local variable or method ‘files’ for main:Object
{% endhighlight %}

Obviously each of the individual bits need to be made into strings. This could be done using <code>method_missing</code>, but that will hijack other uses of the method and potentially mask real failures. What we need is a scope to make it safe to use <code>method_missing</code>. I first saw a way of doing this in the [hobo framework](http://github.com/tablatom/hobo/blob/250a7db560c8a756a53529ae250fb1fdbc10feaf/hobofields/lib/hobo_fields/fields_declaration.rb): we can <code>instance_eval</code> a block in the context of an object which just defines <code>method_missing</code>:

{% highlight ruby %}
class PathConverter < BlankSlate
  def method_missing(method)
    method.to_s
  end
end

def path(&block)
  PathConverter.new.instance_eval(&block)
end

path { files/are/here } # => "files/are/here"
{% endhighlight %}

We inherit from [BlankSlate](http://onestepback.org/index.cgi/Tech/Ruby/BlankSlate.rdoc) to avoid any clashes with methods that already exist on object. All methods are converted into Strings, and then we can use the Merb String method. Now, how about names that start with capitals?

{% highlight ruby %}
path { Users/ohthatjames/Documents } # => uninitialized constant Users
{% endhighlight %}

OK, now we have to capture <code>const_missing</code>s as well. How about this?

{% highlight ruby %}
class PathConverter
  def self.const_missing(const)
    const.to_s
  end
end
path { Users/ohthatjames/Documents } # => uninitialized constant Users
{% endhighlight %}

Hmmm. We're in the context of <code>PathConverter</code>, why doesn't this work? After a bit of googling, there's an [explanation](http://www.ruby-forum.com/topic/101701):

<blockquote>
  <p>All that instance_eval does is set self and execute the block. Constants don't depend on self for their scope; they use a kind of quasi-static scoping, and the Bar in your block is resolved (unsuccessfully) as Bar from the top level.</p>

  <p>You'd need to add your const_missing method to Object, so as to cover the top level.</p>
</blockquote>

Right. This is not going to be pretty. We're going to have to define <code>const_missing</code> on <code>Object</code>. But we can't just completely override it, that'd break Rails and many other programs. A potential solution would be to dynamically redefine <code>const_missing</code> in the <code>path</code> method, then replace it with the old method when we're done. However the simplest thing to do is to use a global variable:

{% highlight ruby %}
module ConstantToString
  def const_missing(const)
    return const.to_s if $convert_to_strings
    super
  end
end
Object.extend ConstantToString

def path(&block)
  $convert_to_strings = true
  result = CodeConverter.new.instance_eval(&block)
  $convert_to_strings = false
  result
end

path { Users/ohthatjames/Documents } # => "Users/ohthatjames/Documents"

{% endhighlight %}

So we set and unset <code>$convert_to_strings</code> around the <code>instance_eval</code> and only convert the constant to a string if <code>$convert_to_strings</code> is true. I can't say I like it, but it works. We also extend <code>Object</code> with a module rather than just redefining <code>const_missing</code> so that we can maintain any other changes made to <code>const_missing</code>.

How about absolute paths?

{% highlight ruby %}
path { /Users/ohthatjames/Documents } # => unknown regexp options - hthatja
{% endhighlight %}

The Ruby parser seems to interpret this as a regular expression. Can we use this to our advantage? 

{% highlight ruby %}
path { ///Users/ohthatjames/Documents } # => NoMethodError: undefined method `/' for //:Regexp
{% endhighlight %}

This is valid Ruby, we just need a / method for <code>Regexp</code>:

{% highlight ruby %}
class Regexp
  def /(o)
    ""/o
  end
end

path { ///Users/ohthatjames/Documents } # => "/Users/ohthatjames/Documents"
{% endhighlight %}

This works, but there are still some things I can't find solutions for: we can't use a Ruby keyword or a filename with spaces. We would still have to put quotes around them. 

So, is it possible to remove the quotes? Yes, mostly. Is it at all practical? Definitely no. It's a glorified string concatenator, achieving something that can be done in much easier, cleaner ways. The hoops we have to jump through for such a questionable endeavour make it hard to sell: overriding both <code>method_missing</code> and <code>const_missing</code>, using a global variable and defining a method on <code>Regexp</code> is just going to make life difficult for anyone wanting to extend the code or for those who find their own code clashing with it.

Still, it's a good example of how the power of Ruby can provide solutions to things that at first seem impossible. But it's an even better example of when to leave certain features of Ruby well enough alone!

Full code:

{% highlight ruby %}
class String
  def /(other=nil)
    return self if other.nil?
    File.join self, other.to_s
  end
end

module ConstantToString
  def const_missing(const)
    return const.to_s if $convert_to_strings
    super
  end
end
Object.extend ConstantToString

class PathConverter
  def method_missing(method)
    method.to_s
  end
end

class Regexp
  def /(other)
    ""/other
  end
end

def path(&block)
  $convert_to_strings = true
  result = PathConverter.new.instance_eval(&block)
  $convert_to_strings = false
  result
end
{% endhighlight %}

