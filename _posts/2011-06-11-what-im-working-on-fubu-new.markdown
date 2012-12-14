---
layout: post
title: What I'm Working On - Fubu New
---

A screencast overview of what I'm currently working on today.


Footage
---

Testing out [screenr][screenr] today.

<div>
<iframe src="http://www.screenr.com/embed/jd8s" width="650" height="396" frameborder="0"></iframe>
</div>


Fubu New
---

I'm currently working on the `fubu new` command. It's a command line utility for
generating a working [FubuMVC][fubu] project. Yesterday, I spent Open Source
Friday working with [Sam Merrell][sam] on determining a strategy for replacing
placeholders in a template project. Sam has done some work with T4, and also
threw around the idea of writing a basic templating engine. However, both seem
a bit overkill for what's actually necessary. `String.Replace()` makes the cut.

At the moment, `fubu new` only works against a directory. However, adding a
default template and simplifying distribution of 3rd party templates (zip, git,
etc), is planned.

Spark
---

As for the issue with Spark, I'll be diving into that as well.
`Spark.Parser.ViewLoader` is spitting out null for one of its values.

Fubu on Mono
---

Not covered in the screencast, but also an effort I've put forth this morning
is attempting to build FubuMVC on mono. It was working at one point and now is
failing on FubuMVC.Deployers

    FubuBottleDestination.cs(49,75): error CS0584: Internal compiler error: Method not found: 'Bottles.Deployers.Iis.ServerManagerExtensions.CreateSite'.

When that method does, in fact, [exist][bottles].

[screenr]: http://www.screenr.com/
[fubu]: http://fubumvc.com/
[sam]: https://twitter.com/smerrell
[bottles]: https://github.com/DarthFubuMVC/bottles/blob/37d123113c158cc5dc095624e48fa04017741009/src/Bottles.Deployers.Iis/ServerManagerExtensions.cs
