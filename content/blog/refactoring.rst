---
title: "Refactoring as an optimization problem"
date: 2021-02-01T12:06:51+02:00
author: "Gustav Larsson"
draft: true
---

.. http://jakevdp.github.io/blog/2012/10/07/xkcd-style-plots-in-matplotlib/

I've been thinking of software development and machine learning.

.. and I'm starting to see refactoring as an optimization problem. 

When performing code reviews, I think it's important to consider what kind of feedback to give, and
what to omit. 
Besides discovering pure bugs, a big part of code reviews is identifying potential issues that can affect maintainability. 
I'm thinking that certain flaws should never be allowed to enter the code base, while
others are ok, if it helps the development velocity.

.. With an existing code base, refactoring is the way we decrease maintenance cost.

Refactoring is the way we decrease maintenance cost of an existing code base. 
**And we can see refactoring as an optimization problem.**  If we are thoughtful, we can make it an easy optimization problem.

.. **Refactoring is an optimization problem. And we can make it an easy or a difficult one.**

If you know where you're going, and the path is `convex <https://en.wikipedia.org/wiki/Convex_function>`_, it's
a relatively effortless activity that is usually worth the time investment. 
We can use the cheap `gradient descent <https://en.wikipedia.org/wiki/Gradient_descent>`_ algorithm . 



{{< figure src="/static/refactoring/plot2.png" width="400px" >}}

If you only have local problems, such as poorly named local variables or unnecessarily complex logic
within a single function, this is easy to fix. 
It's also an isolated problem: If the abstraction that the function provides is still reasonable, the fact
that the implementation is messy shouldn't affect the rest of the application. 
Fully getting rid of these types of problems is of course preferable, but shouldn't be the main goal of a
code review. 


However, if there's a messy relationship and coupling between multiple methods or multiple classes (or multiple systems!), this
should probably be fixed before the change enters the code base. 

.. Cleaning it up after the fact requires much more effort, and it has a tendency of being a problem that accumulates more problems. 


{{< figure src="/static/refactoring/plot1.png" width="400px" >}}

Seen as an optimization problem, we are stuck in an local minimum: We might still be able to make small
improvements, but to make significant improvements, we need to make things worse before they get better. 
This requires more commitment, and is less likely to get prioritized.
Additionally, the hills around this minimum tend to grow by themselves, if left unattended.  

Avoiding getting into this state should be a priority. 


.. Algorithmically, the problem becomes NP-Hard. 

Put in a different way, the code review comments that are easiest to make are the least likely to provide value. 
**A truly valuable review doesn't point towards the nearest local minimum, it peeks over the hill and finds a better one.**
If the checked in code is kept convex, technical debt is less likely to build. 

.. **A truly valuable reivew requires real understanding and identifies a way out of a local minimum**. 


.. it identifies away out of it. 


.. , which of course requires effort. 

We also have to keep in mind that we all have different estimates on how low the minima go. Everyone reading the code might agree that
it's possible to make things better, but not
necessarily on which direction is most likely to lead to the lowest minimum. 

.. To make things more interesting, we can also consider a local maximum (or a `saddle point <https://en.wikipedia.org/wiki/Saddle_point>`_, but I don't want to plot that). 
.. Everyone working on the code will agree that it's possible to make things better, but not 
.. necessarily on which direction is likely to lead to the lower minimum. 

{{< figure src="/static/refactoring/plot3.png" width="400px" >}}
