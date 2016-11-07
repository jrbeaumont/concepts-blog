---
layout: post
title:  "Concepts V: Set-Reset Latch"
date:   2016-07-05 13:03:00 +0000
categories: concepts
---

Set-Reset Latch
---------------

After learning how a [C-element]({{ site.url }}{{ site.baseurl }}/concepts/2016/01/13/c-element.html) can be
produce with concepts, we can introduce other gates. In this case another
fairly standard gate, a *latch*.

The operation of a latch is to hold a given value. There are several different
types of latch, but in this case we will talk about a *Set-Reset latch (SR
Latch)*. This latch has two inputs, one will *set* the latch when high,
causing its one output to be high. The second input will *reset* the latch
when it is high, causing its output to be low.

So lets see how we can specify this using concepts.

Concepts
--------
{{ site.url }}{{ site.baseurl }}/2015/11/24/initial-states.html)
For this, let's start off by referring to the two inputs as `a`
and `b`. The output is `c`.

So, first of all, lets think about how we can set the latch:

{% highlight haskell %}
set = a+ ~> c+ <> a- ~> c-
{% endhighlight %}

And it's that simple. Simply put, when one of the outputs, `a` in this case,
goes high, we want the output, `c`, to go high.

So we have described the interaction for set using one of the inputs and the
output, now lets describe the reset action:

{% highlight haskell %}
reset = b+ ~> c- <> b- ~> c+
{% endhighlight %}

This will cause the output to go low when the second input, `b`, goes high.

These two concepts describe the operation of a SR-latch. But we still need to
think about how the SR-latch will act initially, such as when it first
receives power.

Unless specifically required, we should assume that the output of the latch
should be low at startup. Because of this, we don't want it to be set
initially by `a` being high, so we can set this to 0 initially also.

However, with the second input, `b`, setting this high at this point
would ensure that the output will reset to 0 at start up, but if the initial
state of `c` is already set to low initially, we shouldn't need to set `b` to
high, so we will set the initial state of this to low too.

The
[initial state concept]({{ site.url }}{{ site.baseurl }}/concepts/2015/11/24/initial-states.html)
will be as follows:

{% highlight haskell %}
initialState = initialise0 [a, b, c]
{% endhighlight %}

Now let's create the scenario concept for the SR-Latch:

{% highlight haskell %}
SRLatch = set <> reset <> initialState
{% endhighlight %}

And this can be converted to a Signal Transition Graph, so this
specification can be verified, simulated, tested and then synthesised. The STG
for a SRLatch will be as follows:

![Translated STG of the set-reset latch concepts]({{ site.url }}{{ site.baseurl }}/images/s-r-latch0.svg)

This is a fairly simple STG, but simulation would prove that setting `a` high
will allow `c` to transition high. Once it has transitioned high, it cannot
transition low until `b` has been set high.

For some simpler understanding of how this latch works, we can change some of
the signal names. In this case, we will change `a` to `s`, which stands for
*set*. `b` will be set to `r`, which stands for *reset*. Finally, we can call 
`c`, `q`. `q` is a standard output symbol for latches.

This will change the concepts to the following:

{% highlight haskell %}
SRLatch = set <> reset <> initialState
  where
    set = s+ ~> q+ <> s- ~> q-
    reset = r+ ~> q- <> r- ~> q+
    initialState = initialise0 [s, r, q]
{% endhighlight %}

This will produce the following STG:

![Translated SR-Latch with more standard names]({{ site.url }}{{ site.baseurl }}/images/s-r-latch1.svg)

Changing the names of the signals does not affect the operation.

Now the SR-Latch has been defined, if a user has signals which will be inputs
to a latch, they can use the concept and pass their signals into this, in the
order of `s`, `r` and `q`.  For example someone can use the concept:

{% highlight haskell %}
SRLatch x  y  z
{% endhighlight %}

In this case, `x` will be the set signal, `y` is the reset and `z` will be the
output.

Some reuse
----------

As is usually the case with concepts, we can define a SR-Latch using
previously defined gates as concepts.

First of all, if we look at the `set` concept, it is clear that `s` and `q`
form a
[buffer]({{ site.url }}{{ site.baseurl }}/concepts/2015/09/30/buffer.html).
 So this can be rewritten as:

{% highlight haskell %}
set = buffer s q
{% endhighlight %}

The `reset` concept can also be redefined. In this case, it acts as an
[inverter]({{ site.url }}{{ site.baseurl }}/concepts/2015/10/21/inverter.html):

{% highlight haskell %}
reset = inverter r q
{% endhighlight %}

As the initial state doesn't need to be changed, we can compose these concepts
as above to produce the `SRLatch` concept:

{% highlight haskell %}
SRLatch = set <> reset <> initialState
{% endhighlight %}

Or we can reduce the number of concepts defined and compose these as follows:

{% highlight haskell %}
SRLatch = buffer s q <> inverter r q <> initialState
{% endhighlight %}

There are of course several ways of specifying an SRLatch using concepts, but
these are two of the simpler ways, either using signal-level concepts, or
predefined gate-level concepts. The preference is entirely on the user.

Finally
-------

In this post, we have explained another simple logic gate. The discussion
includes the basic operation being specified as signal-level concepts, or by
taking this a step higher and using pre-defined gates.

We have also shown how while the concept defines certain signal names, these
aren't set. A default SRLatch concept will use these signal names, but if
these signals are not used in a design, a user can pass the desired signal
names into the concept, and these names will replace the default ones.

All concepts described here can be used with the concepts translation tool,
available from <https://github.com/tuura/concepts>, or with this tool
integrated with [Workcraft](http://www.workcraft.org/), for the ability to
visualise the translated STGs. Note, in this post we use `a+` and `a-`,
however the tool is written in Haskell, which does not support postfix
operators. In the tool, `rise a` and `fall a` must be used instead.

This blog series will continue with more standard gates, this time featuring
an interesting type of causality, [***OR-causality***]({{ site.url }}{{ site.baseurl }}/concepts/2016/08/05/or-causality-1.html)
