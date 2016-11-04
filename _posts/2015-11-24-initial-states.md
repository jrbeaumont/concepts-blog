---
layout: post
title:  "Conecepts III: Initial states"
date:   2015-11-24 15:55:00 +0000
categories: concepts
---

Initial States
--------------

Now we have introduced a couple of simple gates,
[a buffer]({{ site.url }}{{ site.baseurl }}/concepts/2015/09/30/buffer.html)
and [an inverter]({{ site.url }}{{ site.baseurl }}/concepts/2015/10/21/inverter.html),
we should now discuss an important part of Signal Transition Graphs, and
their implementation using Concepts. These are *initial states*.

Initial states are used in Signal Transition Graphs to show what state a
signal is in, high or low, when the system being designed starts. This is
important as STGs require initial states for simulation, verification and
synthesis.

In the previous posts, we have assumed that all signals involved in the STGs
are initially set to 0, and this has been indicated by placing a *token* in
the places which are before each signals `+` transition (as above), but this
assumption cannot be applied to every STG.

Example - A Handshake
---------------------

So now let's use a previous example, a Handshake, and discuss what we can do
to ensure there are initial states in an STG produced using concepts.

The concept used to describe a handshake is as follows:

{% highlight haskell %}
handshake a b = buffer a b <> inverter b a
{% endhighlight %}

This concept captures the interactions of signals `a` and `b`, using
previously defined `buffer` and `inverter` concepts, but this does not explain
the state of these signals when the handshake starts.

Initial state concepts can be defined similarly to other concepts, and thus
can be composed with other concepts as normal. For this example, let's say the
two signals involved in handshake are both initially 0.

{% highlight haskell %}
initialState = initialise a 0 <> initialise b 0
{% endhighlight %}

There is some syntax to discuss here:

`initialise a 0` - This is used to express an initial state. The state of the
signal, `a`, will be initially set to 0.

The concept, `initialState`, implies that the initial state of signals `a` and 
`b` will be 0, so the first transition of each can be `+`, from low to high.

Now we can include this concept with the handshake concept:

{% highlight haskell %}
system = handshake a b <> initialState
{% endhighlight %}

Passing this concept into the translation algorithm will produce the same STG
as in the previous post:

![Translated STG of handshake with initial state 00]({{ site.url }}{{ site.baseurl }}/images/buffer-inverter-00.svg)

And this can be resynthesized to produce the same handshake STG:

![Resynthesized handshake with initial state 00]({{ site.url }}{{ site.baseurl }}/images/handshake-00.svg)

However, what if the initial states are not both 0? For this, let's redefine the initial state concept and compose this with a handshake concept:

{% highlight haskell %}
initialState = initial a 0 <> initial b 1
system = handshake a b <> initialState
{% endhighlight %}

Now, `a` should initially be 0, and `b` should initialy be 1. The STG produced from these concepts is:

![Translated STG of handshake with initial state 01]({{ site.url }}{{ site.baseurl }}/images/buffer-inverter-01.svg)

This is the same as the previous version, but the tokens have shifted. After resynthesis, the STG is as follows:

![Resynthesized handshake with initial state 01]({{ site.url }}{{ site.baseurl }}/images/handshake-01.svg)

Again, very similar to the previous post-resynthesis STG. But in this case, we can see that `b-` is the first transition, instead of `a+`.

Initial States in the Translation Tool
--------------------------------------

In the [concepts translation tool](https://github.com/tuura/concepts), there
is 2 ways of describing an initial state. In many cases, it is
inconvenient to use `initialise a 0` for every single signal, for example, if
there are a large number of signals, then you will have a long list of initial
state concepts, which can make the concepts file larger, and more complicated
to understand.

To help with this, we have included concepts to initialse multiple signals to
the same initial value at the same time. These are:

{% highlight haskell %}
initialise0 [a, b, c, d] <> initialise1 [x, y, z]
{% endhighlight %}

These concepts take in a list of the signals in the system, comma separated
within `[ ]` brackets. The concept `initialise0` will set the initial state of
each signal listed to 0. Conversely, `initialise1` will set the initial states
of all the signals listed to 1.

As with every other concept, these can be composed with other `initialise`
concepts, as well as any other concepts.

*So, what if a set of concepts doesn't contain any initial state concepts?*

An STG with no initial states defined is not a correct STG, it violates
properties of STGs which are necessary in order to be used with tools which
perform useful functions such as simulation, verification and synthesis, for
example.

If any signal or signals are not initialised, then the concepts translation
tool will fail, and present an error to a user, listing the signals without
initial states.

This will reduce any errors in the translated STGs, by either producing an STG
which has no initial states, which cannot be used further without having
initial states, or by the tool assuming initial states without user input,
which may cause an erroneous STG to produced.

Finally
-------

In this and the previous two posts, we have covered the basics of concepts,
and the design process of various simple gates.  Now we can move onto
discussing some more complicated designs, and how we can use pre-existing
concepts, and define new ones.

The next blog post will be Concepts IV: A C-element. This will be the first in
a set of posts describing some standard logic gates, and from this we will
work up to some more complex examples, using some of the previously described
gates.

As with all these posts, the concepts discussed in this post can be used in
conjunction with the concepts tool, available from
<https://github.com/tuura/concepts>, or from within
[Workcraft](http://workcraft.org), where the concepts tool is built in.

For clarity, in this post we have referenced signal transitions in the form
`a+` and `a-`. The tool however doesn't support postfix notation, and as such,
`a+` and `a-` must be replaced by `rise a` and `fall a` respectively in order
to be used with the tool.
