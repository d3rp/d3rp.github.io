---
title: "Zappa Pipenv"
date: 2018-04-01T10:29:31+03:00
draft: true
---

Zappa.. I have one poster on my wall, and it's of Frank Zappa. It was obvious to me, as was python's name, what were the origins of these names, but I was hesitant to believe it would be so - surely no-one named their projects with arbitrary musings of outer-CS topics.. But as python was derived from the English tv sketch series, so was Zappa derived from Frank Zappa - and both did something totally unrelated to both of their name origins. Hooray - you just lost a minute of your life
reading this uninformative "introduction". Let's amp it up with more blog-gold - need more substance, as internet is so lacking of content..

Zappa is quick Lambda-launcher for AWS. Codifying deployments seems to be easy with it. Coming from a cloudformation, security focused, CI/CD-Certificates-For-Everyone background that sounds like a fairytale.

Pipenv is a new introduction to the saga of python packaging implementations. It handles virtual environments while pipping your packages in place. Pipenv's man page sais:

"Pipenv is a tool that aims to bring the best of all packaging worlds (bundler, composer, npm, cargo, yarn, etc.) to the Python world. Windows is a first–class citizen, in our world."

How cute: Windows is a first-class citizen. I mean Microsoft is taking steps in the opensource community being the far and largest contributer in github if counted in projects and commits, but - even if it's going to bite me in the future for saying this - its closed source intentions to create awkward graphical tools like they think everything they make is the next silver bullet... first-class citizenship?

Install pipenv:

{{< highlight bash >}}
pip install --user pipenv
{{< / highlight >}}

Create a new environment and install zappa:

{{< highlight bash >}}
mkdir zappa_one && cd zappa_one
pipenv --python 3.6
pipenv install zappa
{{< / highlight >}}

Run the environment to start developing:

{{< highlight bash >}}
pipenv shell
{{< / highlight >}}

What have we done so far is..
