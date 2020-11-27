---
title: "Experiments & Architectural Process"
date: 2020-09-13T18:09:50-04:00
---

It sometimes takes working on an application with a *bad* architectural design to realize that you never want to work on such a project again.
But, *how* does a programmer make sure their application doesn't become *that project*?

<!--more-->

Design patterns are useful because they give a name to a complex set of logic that crops up in design.
Take the factory pattern for example - you could easily create a sound class building concept without ever mentioning the word 'factory', but learning about the factory pattern will give you a mental roadmap for how the code *should* look.
That context can help reduce the cognitive load of building (or maintaing) a code because developers apply their prior knowledge to that chunk of code.
But, for this to work, developers need that prior context to gain that common understanding.

There are a number of ways to get this experience design-pattern experience -- courses, books, conference talks -- but I have found that only way for me to *really* understand an abstract design pattern is to use it in real code.
Someone like me with few life-related responsibilities can maybe get this broad experience from weekend projects, but weekend projects aren't feasible for everyone.
So, an organization striving to improve their development team would need to provide opportunities for *breadth* in technology experience.
This is most easily accomplished with **experiments**.

## What is an experiment (in software)?

Experiments can be conducted by building small, non-critical components using a different approach.
Unliike traditional development, the feature is *not* more important than the process.
Rather, the goal of the experiment is to to expose the team to a new software pattern, process, framwork or some other new technology.

This way, the next time a problem of real business value needs to be solved, the team will have a wider area of experience to make more informed architectural decisions.


