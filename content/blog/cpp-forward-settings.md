---
title: "Intuitive settings class from cpr in C++"
date: 2021-01-30T20:14:30+02:00
draft: true
author: "Joni Turunen"
---

tl;dr: How to fight the formation of spaghetti code, when designing a configuration/settings
class, and make it feel intuitive.

I was toying around with my hobby project and came across a wrapper library for
libcurl. At first I was a little annoyed with its SSL handling and its seemingly
complicated way of setting it up. As I figured out how the configuration parameters
were supposed to be inserted in order to get things going, its ways seemed actually
pretty useful. Also I noticed, that apparently there's such a thing as a variadic 
std::forward.

A little context before digging deeper into this. What we have here,
is a Settings-class, that can be configured in various ways. The concrete
examples shown below are toy examples, but what I was most excited about was how you could
instanciate and control the options on the "client side" of the code without too
much messyness - and still provide an intuitive way to lay out the options for
that Settings-object. In other words, it allowed a lot of control over the
option configurations while remaining reasonably simple to use and without
being too complicated to maintain.

For example, if we have a Settings-object with some context, we can set it up 
with the use of some instanciated classes acting as options (here `Foo` and
`Bar`):

{{< highlight cpp >}}
Settings s(Foo(), Bar());
{{< / highlight >}}

This allows to configure those options quite freely by using their
initialization logic as we could configure `Foo()` in many ways and passing
the instance to the `Settings s`.

In order to get this going create a Settings class:

{{< highlight cpp >}}
#include <string>

struct Settings
{
    std::string ctx
}
{{< / highlight >}}

Just to make this as simple as it needs to be in order to see its potential,
let's have an option object with some context that we use to manipulate the
Settings' context:

{{< highlight cpp >}}
#include <string>

struct Option
{
    std::string word = "default ";
}
{{< / highlight >}}

Then adjust Settings class to use that "word" that was
defined above:

{{< highlight cpp >}}
struct Settings
{
    std::string ctx;
    
    void set_option(Option&& option)
    {
        ctx += option.word;
    }
}
{{< / highlight >}}

Lets start with the metaprogramming by creating a getter function, which
takes the instanciated options, creates a Settings object and uses the options
by passing them through the "set_options pipeline":

{{< highlight cpp >}}

template <typename... Ts>                               // 1
Settings get(Ts&&... options)                           // 2
{
    Settings s;
    priv::set_options(s, std::forward<Ts>(options)...); // 3
    
    return s;
}

{{< / highlight >}}

// 1
Collecting all typenames - variadic typenames - with "typename...".

// 2
Ts&&... - variadic move reference meaning there's one or more of "options" and
they can be of different types, because of the variadic typenames.

// 3
Using a priv namespace for convenience. Passing the newly created Settings
object and forwarding all of the options with `std::forward<Ts>(options)...`.
This had a few surprising features. 

- First, the forward allows types to be
infered i.e. in the end the implementation can take into account just one base
level interface ("Option") when the object is forwarded.

- Second, the syntax for this variadic pass through is
not all that obvious - "options" is an initializer list, for which the types are
collected in Ts and paired with the variadic "..." notation.

Next let's define the last missing piece in order to glue the
Settings.set_option and the getter. Here we'll use something I like to think as
recursive metaprogramming. I call it that, as we'll define a base case `template
<typename T>` and supplement that with a `template <typename T, typename... Ts>`.
And in this second step we snatch the first typename as a single T and the
remaining once with `... Ts` hence using the preprocessor for a recursion.

{{< highlight cpp >}}

namespace priv {
    template <typename T>                           // 1
    void set_option(Settings& s, T&& t)
    {
        s.set_option(std::forward<t>(t));           // 2
    }

    template <typename T, typename... Ts>
    void set_option(Settings& s, T&& t, Ts&&... ts) // 3
    {
        set_option(s, std::forward<T>(t));
        set_option(s, std::forward<Ts>(ts)...);     // 4
    }
}

{{< / highlight >}}

// 1
Base case. When there's a call for set_option(s, t), this is used. There's
such a call before // 4.

// 2
Here we use the `Settings.set_option(...)` we defined earlier. This is the
main piece of glue. All the other metaprogramming tricks shown here are there to
funnel one option at a time into this call.

// 3
Important to collect first T alone and collect the remaining ones in Ts.
Again, in order to funnel a T at a time to `set_option(s, t)`

// 4
The remaining `Ts` are looped back (recursive) into // 3 thus separating
the first T and collecting the others into Ts. Like with recursive functions,
this continues until there's nothing left for Ts in which case only // 1 is
called.

There you have it. Now to complete this, we can make other options and a main
function to try them out:

{{< highlight cpp >}}

struct Foo : Option             // 1
{
    Foo() : Option("foo ") {};  // 2
}

struct Bar : Option
{
    Bar() : Option("bar ") {};
}

void main()
{
    Settings s(Foo(), Bar());   // 3
    std::cout << s.ctx << "\n"; // 4
}

{{< / highlight >}}

// 1
Like I mentioned earlier, `std::forward<T>(t)` and `T&& t` allows to pass
through infered types from `Option`.

// 2
Just for the sake of demonstrating this, the `Option` had a field named
`word`, that we concatenated to `Settings.ctx`. As we've used structs, we can
just define the field to simulate this options-manipulating-settings-object
behaviour.

// 3
This is what intrigues me: if we had more elaborate options and
configuration options for settings, here we could define them when instantiating
`Foo` and `Bar`. There's no need to mangle multiple definitions and have a
trickle down if-else labyrinths in some manager, but instead define the logic in
the options themselves, and open up the user side of the code here - as the
CTOR.

// 4
Printing the wonderful results of this experiment. It should read now: "foo
bar"

This is to demonstrate, that we did in fact pass the `Foo` and `Bar` to the
`Settings` object, which "used them to set up itself". While passing through the
Options, the metaprogramming resolved the types according to the Options. 

I'm happy if you've endured my ramblings this far, because thus far this hasn't
been a major deal. The settings class demonstrated here within doesn't deviate
much from any other class, that uses an interface as the type in the parameters
to use inherited features. However, this pattern is nice, because aside from
that it lends to individual classes to be used as well, that can be all unique.

For this last extension, let's consider we have individual classes such as:

{{< highlight cpp >}}
class Foo;
class Bar;
{{< / highlight >}}

And these don't inherit from anything. Just for the sake of an example, imagine
those have implementations such as:

{{< highlight cpp >}}
class Foo
{
    Foo(std::string name = "shady")
    {
        std::cout << "Chiki Chiki slim " << name << "\n";
    }
}
{{< / highlight >}}

Then we can use parameter overloading to configure all the cases for settings
as:

{{< highlight cpp >}}
struct Settings
{
    std::string ctx;
    
    void set_option(Foo option)
    {
        ...
    }
    
    void set_option(Bar option)
    {
        ...
    }
}
{{< / highlight >}}

Now, it didn't do much in this example, but I feel this can lead to cleaner code
when using this pattern.  Such a settings class could have been configured with
the use of for example Enums. However, I've found that that often trickles down
to an if-else spaghetti part - maybe a manager or a factory - and the
cleanliness of the code lies on the good-will of the developer next in line to
make changes to the code - and not everyone feels like a boy-scout when in a
hurry. In contrary to that, here we use metaprogramming to help a little with
the process, as this enforces defining every option in their own class and every
set_option variation in their own overriding method.

My inspiration for this was cpr library for wrapping libcurl in C++
https://github.com/whoshuu/cpr . More specifically, the `cpr::Get(...)` uses
this pattern.
