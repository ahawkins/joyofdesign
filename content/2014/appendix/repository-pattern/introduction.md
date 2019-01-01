---
type: docs
bookShowToc: false
---

# Introduction

The repository pattern changed how I approached writing software and
it's key point in the joy of design series. It helped me improve from
writing poorly constructed web applications. I'm not perfect
now, but I'm a hell of a lot better. I wrote my own little MySQL
adapter for my php apps so I could query the DB wherever I wanted to.
Then came Rails with its wonderfully power implementation of the
Active Record pattern. Rail's ActiveRecord made me realized the power
of abstractions over data. Gone where the day of sanitizing forms and
writing SQL. Things had leveled up. But something was missing. There
was never any true single responsibility. Every application was working
under the assumption that a model is data plus persistence. Rails only
made that worse and has probably scared many people for life. The
repository pattern is the anti-active record pattern. It enforces true
separation of concerns by design. It encourages to separate how you
think of your objects. It is a decoupled state of mind. Once you enter
this zen state you cannot go back.

The repository pattern is not a silver bullet. Proper use will make
your software's architecture more sound, but it will not write your
application for you. One simply cannot just "use" a repository as
you'll come to see through this series of posts. I haven't found a
good implementation in Ruby so I worte my own inside
[Chassis](https://github.com/ahawkins/chassis). All the posts assume
this implementation, so these posts also serve as some documentation
for people interesting in using the library. I hope this appendix
makes the repository pattern feel more approachable for you.
