---
layout: post
title: Reference vs. Value Comparisons
date: 2018-03-08 16:17:20 -0800
description: References and optimizations in the Python interpreter
img: python.png
tags: [Python, Programming]
---

According to the [PEP 8] python style guide, comparisons to singletons should be done using `is` or `is not`. For example, when checking for the existence of a key in an object, the proper way to do so would be `if obj.get('key') is None`. That being said, this is only a style guide. Python will still happily compare singletons with equality operators such as `==`.

{% highlight python %}
>>> obj = {}
>>> obj.get('key') == None
True
>>> obj.get('key') is None
True
{% endhighlight %}

Using `is` will be faster, because it checks for the reference rather than evaluating the variable. But what about non-singletons? It would appear that reference comparisons work for integers too.

{% highlight python %}
>>> x = 1
>>> x == 1
True
>>> x is 1
True
{% endhighlight %}

So now you're thinking: _Does that mean if I change all the comparisons in my python code to reference comparisons, my code would be marginally faster?_

Let's see a few more samples first.

{% highlight python %}
>>> x = 1000
>>> x == 1000
True
>>> x is 1000
False
{% endhighlight %}

Apparently, for high integers, the reference comparison no longer returns `True`. Why?

{% highlight python %}
>>> x = 1000
>>> y = 1000
>>> x is y
False
>>> x = 1
>>> y = 1
>>> x is y
True
{% endhighlight %}

When `x` and `y` are both assigned a value of 1000, they are referring to different locations in memory. This makes sense, as most variables behave this way. What doesn't make sense is why `x is y` holds true when they are set to a value of 1.

As it turns out, Python actually initializes integers -5 through 256 as singletons. This is done for performance reasons. The speedup occurs, for example, when you perform `for i in range(100)` and start using `i` in the loop.

Although Python is frequently referred to as a rather slow language, there are more optimizations than meets the eye. Can you think of why the following happens?

{% highlight python %}
>>> x = 'abc'
>>> y = 'abc'
>>> x is y
True
>>> x = ['abc']
>>> y = ['abc']
>>> x is y
False
>>> x = 1000
>>> y = x
>>> x is y
True
{% endhighlight %}

[PEP 8]: https://www.python.org/dev/peps/pep-0008/#programming-recommendations
