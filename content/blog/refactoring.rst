---
title: "Refactoring as an optimization problem"
date: 2021-01-30T12:06:51+02:00
author: "Gustav Larsson"
draft: true
---

.. http://jakevdp.github.io/blog/2012/10/07/xkcd-style-plots-in-matplotlib/

I've been thinking of software development and machine learning.

When performing code review, I think it's important to consider what kind of feedback to give, and
what to omit. I'm thinking that certain flaws should never be allowed to enter the code base, while
others are ok, if it helps the development velocity.

**Refactoring is an optimization problem. And we can make it an easy or difficult one.**

If you know where you're going, and the path is `convex <https://en.wikipedia.org/wiki/Convex_function>`_, it's
a relatively effortless activity that is usually worth the time investment. 
We can use the cheap `gradient descent <https://en.wikipedia.org/wiki/Gradient_descent>`_ algorithm . 

If you only have local problems, such as poorly named local variables, or unnecessarily complex logic
within a single function, this is easy to fix. 


{{< figure src="/static/refactoring/plot2.png" width="400px" >}}

It's also an isolated problem: If the abstraction that the function provides is still reasonable, the fact
that the implementation is messy shouldn't affect the rest of the application. 


If, however, there's a messy relationship and coupling between multiple methods or multiple classes (or multiple systems!), this
should probably be fixed before the change enters the code base. 
Cleaning it up after the fact requires much much more effort, and it has a tendency of being a problem that accumulates more problems. 


{{< figure src="/static/refactoring/plot1.png" width="400px" >}}

Seen as an optimization problem, we will get stuck in an local minimum: We might still be able to make small
improvements, but to make significant improvements, we need to make things worse before they get better. 
This requires more commitment, and is less likely to get prioritized.  


.. Algorithmically, the problem becomes NP-Hard. 



To make things more interesting, we can also consider a local maximum (or a `saddle point <https://en.wikipedia.org/wiki/Saddle_point>`_, but I don't want to plot that). 
Everyone working on the code will agree that it's possible to make things better, but not 
necessarily on which direction is likely to lead to the lower minima. 

{{< figure src="/static/refactoring/plot3.png" width="400px" >}}
