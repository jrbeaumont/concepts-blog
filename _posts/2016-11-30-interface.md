---
layout: post
title:  "Concepts VII: Interface"
date:   2016-11-30 10:17:00 +0000
categories: concepts
---

Interface
---------

In the previous posts from [this series]({{ site.url }}{{ site.baseurl }}/),
we have discussed the algorithm for translation of concepts, and the concepts
that can be included. The last two-part post were a discussion of
OR-causality, and the special differences needed to allow for translation of
this.

This post discusses the final concept necessary for any form of correct
concepts specification. Without this, a specification will work in a few
cases, but further changes will need to be made to a translated specification
before it can be used in other processes, and as such, we have required this
concept for translation to occur. This is *interface*.

Interface is the description of how the scenario being designed interacts with
the outside world. "The outside world" is a big place, and can refer to pretty
much anything, but in this case we mean what the signals interact with, be it
a user, another scenario, another circuit, or even within this scenario itself.
This is done by describing the *signal type*.

The three types of signal are; Input, Output and Internal. Depending on the
type a signal is defined as, it provides us with an understanding of how this
scenario or circuit specification interfaces with the outside world:

**Input** - Signals that cause changes in circuit being specified. These can
come from other circuits or scenarios, either specified as concepts or not,
user inputs or any source which causes a low-to-high or high-to-low transition
in a signal.

**Output** - Signals which cause changes in *outside* circuits. Output signals
can effectively become input signals to other circuits, used to drive
operations in that circuits.

**Internal** - These are signals which are generated by the circuit being
specified, and used by it too, not coming from the outside world, and not
being used to drive anything in the outside world. While this doesn't
necessarily mean it is part of an interface, as we describe it as how it
interacts with the outside world, internal signals are very important, as we
will discuss later.

Let's discuss the interface concepts, how they work, and some simple examples.

Interface concept
-----------------

Let's jump right in and display these concepts:

{% highlight haskell %}
interface = inputs [a, b] <> outputs [x] <> internals [p, q, r, s]
{% endhighlight %}

This single line specifies the signal types of 7 different signals. `a` and `b`
are inputs, `x` is a single output, and `p`, `q`, `r` and `s` are all internal
signals. This line itself is not likely to be used in a specification, but it
does well to display how the tool deals with the inputs. As long as they are
listed in the forms above, `inputs`, `outputs` and  `internals` can specify
the respective types of any number of signals.

And this leads us on to the requirements of
[the tool](https://github.com/tuura/concepts/).
As the type of one signal can change the effect of the circuit being
specified, it is important that interfacing is done correctly in the concept
specification, and thus, the tool requires that each signal has it's type
declared in order for translation to take place. If a user forgets to specify
a signal's type, it will inform them with a helpful message.

Interface intricacies
---------------------

The
[initial state concept]({{ site.url }}{{ site.baseurl }}/concepts/2015/11/24/initial-states.html)
allows a user to specify a signals initial state more than once. Providing the
initial state that is being specified is the same every time, it causes no
issue. However, if a signals initial state is specified to be both `0` and `1`,
then this causes an error in the translation tool, due to an uncertainty as to
what state the signal should be in. An initial state can cause a major
difference in the operation of the circuit depending on which way it is set,
so this throws a handy error message, informing the user of the problem.

Similarly, a concepts specification can allow a signals type to be specified
more than once:

{% highlight haskell %}
interface = inputs [a, b] <> outputs [b, x] <> internals [a, q]
{% endhighlight %}

In this example, `a` and `b` are stated to be inputs. A signal `x` is stated
to be an output, as well as the previously used `b`. `a` is then stated to be
an internal signal again, as well as a new signal, `q`.

Bascically, at this point, `a` and `b` could be one of two different signal
types. But this will not stop the translation process, providing an error
message. It will still complete, but what interface will these signals belong
to?

To decide this, we have created a hierarchy, consisting of four different
states. From lowest to highest:

**Unused** - This is when a signal has not had it's type delcared. This is so
we know which signals need to have their type declared for an error.

**Input** - An input is the lowest state of declared signal type. Inputs,
while we can specify their behaviour, a circuit will have no direct bearing on
them, and so we cannot directly control them, they are simply signals from the
environment that we control the reaction to in the circuit.

**Output** - Outputs are what we are mainly interested in. The output of a
circuit causes changes in the environment, so a circuit specification must
specify what combination of inputs cause output signals to change. This is why
they are higher than an input in this hierarchy.

**Internal** - Internal signals are similar to outputs, in that the specified
circuit reacts to the inputs to produce both internal and output signals. The
internal signals can also be used to influence the output signals however.
They can be thought of as output signals, but with no output terminals. This
makes them more important than both input and output signals.

But what does this mean for the signals `a` and `b` from the above example?
Well, if a signal is declared as an input, and then later on re-declared as
either an output or an internal, it's type will change this value. Similarly,
if an output signal is redeclared to be an internal signal, this signal will
now be an internal.

However, if you declare an internal signal, and then re-declare its type to be
input or output, it's type will remain as internal. And similarly to this,
declaring an output signal, and re-declaring this as an input will cause it to
remain as an internal signal.

So, since `a` is declared as an input, *and* as an internal, it will be
translated as an internal.

`b` is declared as both an input and an output, and as such will be translated
as an output.

Translation
-----------

So, let's take a simple example, just to see what happens with translation. We
will assume that all signals types are declared. The example we will use is a
C-element. Similar to the the
[post on c-elements]({{ site.url }}{{ site.baseurl }}/concepts/2016/01/13/c-element.html)
the concepts are as follows:

{% highlight haskell %}
CElement = buffer a c <> buffer b c <> initialState <> interface
{% endhighlight %}

This includes an initial state concepts, and the interface concept, but these
are yet to be defined. Let's define them as follows:

{% highlight haskell %}
initialState = initialise0 [a, b, c]
interface = inputs [a,b] <> outputs [c]
{% endhighlight %}

The translated STG will look like this:

![C-element with an interface concept defined]({{ site.url }}{{ site.baseurl }}/images/interface_c-element_1.svg)

This looks the same as the C-element in the dedicated post, with two inputs,
`a` and `b`, represented by their signals being coloured red, and a single
output `c`, coloured blue.

Now let's changed the interface concept to see what occurs. Let's keep all
other concepts the same.

{% highlight haskell %}
interface = inputs [a] <> outputs [b] <> internals [c]
{% endhighlight %}

The STG translated from these concepts will look as follows:

![C-element with a different interface]({{ site.url }}{{ site.baseurl }}/images/interface_c-element_2.svg)

`a` has been defined as an input again, and thus it remains blue. `b` is now
declared to be an output, and so it is blue. `c` is now an internal signal,
which in [Workcraft](http://workcraft.org), is represented by the signal being
coloured green.

Now let's try a slightly more confusing example:

{% highlight haskell %}
interface = inputs [a, c] <> outputs [a, b] <> internals [b, c]
{% endhighlight %}

Each signal in the C-element is declared as two different types. Can you work
out what each type will work out as?

![C-element with a confusing interface]({{ site.url }}{{ site.baseurl }}/images/interface_c-element_3.svg)

Since `a` was declared as both an input and an output, the hierarchy states
that it will be an output. `b` is declared as an input and an internal, and
thus will be translated as an internal signal. `c` is also defined as an
input, and an internal, and thus becomes an internal signal after translation.

Finally
-------

Interface is the final part of the tool that is required. This allows a user
to choose the type of the signals which are part of their concepts
specification.

From here on in, all concept specification posts will include an interface
concept, in order to be as close as possible to the true code requirements for
the tool.

However, to be more clear in these posts we use `+` and `-` to represent high
and low transitions respectively, for example `a+` and `a-`. This cannot be in
the tool however, as it is written in Haskell which does not support postfix
notation. In the tool you must use `rise` and `fall`, for example `rise a` and
`fall a` in order to show high and low signal transitions.

The tool is available stand-alone from <https://github.com/tuura/concepts>,
which includes a manual and some examples. It is also available as part of
Workcraft (<http://workcraft.org>), which automatically produces a visualised
STG. Feel free to use the tool in any form with these, and any other examples
featured in the blog.

This series will continue with some more interesting, lesser-known circuit
types, to attempt to test the limits of concepts.