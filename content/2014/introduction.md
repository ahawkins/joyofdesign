---
type: docs
bookShowToC: false
---

# Introduction

The last year and half have been a transitionary time. I moved from
working for a small startup with only another developer to a larger
full time roller inside a larger team with a far older codebase. My
approach to software design, maintainability, and implementation
changed radically. The changes have been so beneficial that I can not
revert them. That path only leads to anger, anger leads to hate, and
hate leads to the dark side. Some readers may consider my ideas
controversial. I think that's because so many Ruby programmers have
been spoon fed coupling and destructive programming choices since they
started writing Ruby.

I've shown people my techniques and have been met with visceral
reactions like: "What is this?", "Why are they are so many classes?",
"Why don't you just use _insert gem here_?" These are honest
questions but the reactions are more valuable. Some people are
offended. It's a shame that the design triggers these reactions. On
the other hand it's wonderful because something is working. Ideas are
being challenged and people are beginning to think differently (if not
for a moment).

I am not the first person utilize design patterns and
boundaries. I'm sure there are thousands of posts just like this.
Some lessons are best learned through first hand experience. One can
read about design patterns and boundaries until you're blue in the
face. Full comprehension comes from understanding their role in
preventing painful software design--the kind that cause frustration,
misdeadlines, and increase toil.

I'm a web developer exclusively. I prefer working on JSON APIs but
I've also worked on (from what I can see the biggest is the Ember.js
app: [Radium CRM](http://radiumcrm.com)). I've been doing more
full-stack/user-facing stuff since starting my new full-time job.
However writing web services or libraries is 90% of the work. It's a
welcome break from bog-standard Rails web applications. I did enjoy
that at the time since I was shipping and building applications.
Eventually I moved more into APIs and slowly realized I liked this
world far more than the other. Why? The answer is simple. Everything
is boundaries.

All good design enforces strict separation of concern though
boundaries. Boundaries are boxes that encourage design through
protocols. What actually created the boundary for a web service? The
internet itself. When you design a web service, you approach it from
the client or server side. There is only one thing between them: the
data sent between each. There is nothing else. The idea was
liberating. It started a chain reaction which changed everything for
me. I began approaching each aspect of the application from a
different perspective.

The knowledge cannot be expressed in a single post. I decided to
approach this problem in a different medium. This is my first stab at
writing a technical paper. I user paper loosely. This is certainly not
a book nor a published technical work but it's certainly more than
just a post. It started out as a single 20,000 word document and
eventually morphed into this. I was luckly enough to get others to
review my draft. Avdi Grimm was gracious enough to lend his time.
Lucky for me because I respect the hell out of guy. One of his
comments hit home:

> I haven't even reached the need for Repository and Query yet, and
> I've already experienced major design epiphanies. If there's a point
> to all this, it's this: consider not glossing over the building
> blocks that go underneath Repository. Not every app needs a
> repository (some don't even need Mapper). And every single layer of
> the cake, if approached mindfully and intentionally, can bring
> serious benefits.

Continuously layering design patterns changes everything. It did for
me. My first change drawing a three layer boundary common to most
applications: the delivery mechanism, the domain, and persistence. We
begin with delivery mechanisms.
