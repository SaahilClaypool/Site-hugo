---
title: "Ubiquitious Language - 'Software Architect's Handbook'"
date: 2020-07-13T18:09:50-04:00
---

Ubiquitous Language and the imortance of just *agreeing* on what to call things.

<!--more-->

## Ubiquitous Language

Ubiquitous language might not seem like the flashiest thing to pul out of a software architecture book, but I have found that clearly communicating software requirements has been the distinguishing factor of recent projects I have worked on.

To take a step back, a peice of software is only useful in the way it performs some task (or helps a human perform a task).
The details for how it gets that task done are not nearly as important as the simple question: does the software do what it needs to do.
Within a business, 'thing' the program needs to do is often determined in part business owners, and business owners don't always speak the language as the developers.
One solution to this problem is to create a *ubiquitous language*: a common language all team members and stakeholders agree upon to refer to parts of the domain.

Take a recent project I worked on as an example.
My team was tasked with creating what our product team referred to as a 'subscription model' for students.
Unlike many models, the payments made by the students would not follow a monthly cadence, but rather be billed to each student by their school semester.
Moreover, the students would not be paying for indefintely for continued access, but rather just a set number of times before they 'owned' the product.
Does that sound like a 'subscription'? No. No it is not so much a subscription as a payment broken into a series of installments, and it took our engineering team a long time (weeks!) to reconcile that this 'subscription' was really a payment quantity split into irregular installments.

This miscommunication went both ways - internally, or engineering team referred to these payment periods as 'semesters' as to us, each payment period aligned with a school semester.
This colloquial usage led us to name our models, routes, and text within pages as 'semesters'.
It wasn't until we met a month or so later with our training team (responsible with educating school-personel about our software) that we realized the err of our ways - 
'semesters' is not the term all schools use to descibe their course run times.
Moreover, our 'payment periods' would not always line up with these 'semesters' or course blocks.
Long story short, we needed to rename all our end-user facing terminology, but we still had all the dangling references to the term 'semester'.

All of this *may* have been avoided if we had just agreed on what we would call things from the start while we - A ubiquitious language.
