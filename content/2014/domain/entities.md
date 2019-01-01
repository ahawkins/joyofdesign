---
type: docs
---

# Entities

The "Joy of Design" series has covered a lot of ground so far. There is
still a bit more to go. I've covered delivery mechanisms, forms, and
use cases. I haven't talked about the actual business objects
yet. Use cases collaborate with different objects to do things. It's
time to talk about them. They are entities. Entity is a board term on
purpose. An "entity" can be anything really. These classes represent
business concepts. You cannot pigeonhole them into models. There are
more roles than just model. Most importantly they are the nouns in
the applications--the concepts an application is all about.

Consider a CRM. There are many different entities. `Customer`
jumps out right away (it _does_ stand for Customer Relation
Management). There will be many more. `Company`, `Account`, `Deal`, or
`Invoice` to name a few. A blog has `Post` and `Comment`. A
classifieds site has `Ad`. I consider these models because they map
cleanly to business objects. Then there are things that don't follow
the same rules. There are things like `Twitter` for posting tweets.
You might consider this a service object with something like
`TweetService`. Then there are other things likes validators. They are
not models but used by some use cases The point is that there
are many different kind of entities. Let's start off by covering
models.

## Model Classes

First, a word of warning. I want you to think hard about what you
consider a model. Is a model a `Struct` like class encapsulating logic
or is a more complicated object given to you by an ORM? If you sided
with former you're thinking in the right direction. If you're already
thinking about ORMs and persistence you need to stop that. This post
has nothing to do with persistence or storage at all. That is not
important. Models are not persistence and persistence does not make a
model.

Model classes are deceptively easy to implement. Let's create a
`Customer`.

```ruby
class Customer

end
```

Done. Well at least with the first step. A customer will have some data
with it. This is also deceptively easy to implement.

```ruby
class Customer
  attr_accessor :given_name, :family_name
end
```

What if there is more complicated data like addresses? This one
is hard.

```ruby
class Customer
  attr_accessor :given_name, :family_name
  attr_accessor :addresses
end
```

I hope you get the idea by now. Model classes are just standard
classes. There is no magic in them at all. They have accessors just
like anything else. Add methods to encapsulate behavior.

```ruby
class Customer
  attr_accessor :given_name, :family_name
  attr_accessor :addresses

  def should_contact?
    if !recent_email? && purchased_product?
      true
    elsif !purchased_product? && opportunity?
      true
    elsif lead? && received_follow_up?
      true
    else
      false
    end
  end
end
```

I took the liberty of assuming methods were implemented. I think the
example illustrates the point. Classes encapsulate behavior. Just go
ahead and write them. It may seem scary at first but will pay
dividends. I can't express how _uncomplicated_ it is to create these
objects. They have `attr_accessors`. Some methods are public. Some are
private. They have custom `initialize` methods. Some may have inner classes.
Some may even include modules. You know, they are Ruby classes.
Shocking right?

_NOTE_: One person asked me about validations. Entities do not
validate data coming from the outside world. Form objects do that.
The use case collaborates with entities. New entities are created
using attributes from the form. Only check the data in one place:
the border where it enters the system. There is no point to duplicate
logic by validating it another place.

## Service Objects

I'll probably get some flack on the naming but such is life right?
This is the hardest problem in computer science. Here's my working
definition: a service object represents a context-free interaction
with the outside world which may have multiple implementations. Here
are some examples from my past projects. `PushService`. Accepts
`PushNotification` instances and sends them out using some sort of
push service (APN, GCM, WebSocket, whatever). `TweetService` posts
tweets to twitter. `EmailService` for delivering emails. All these
objects exhibit a quality: they need to behave differently in tests.
It does not make sense for tests to tweet or send emails. I design all
my service objects implementation free. The main class delegates all
actual work to an implementation object. The implementation is
switched during development/test/production. The methods are exposed
at the class level. Since these are context free interactions there is
no need to instantiate an object. Here's what they look like:

```ruby
class PushService
  class << self
    def implementation
      @implementation
    end

    def implementation=(impl)
      @implementation = impl
    end

    def push(notification)
      implementation.push notification
    end
  end
end
```

That's all there really is to it. I usually have at least three
implementations: the real one, a fake implementation for tests, and a
null implementation. I prefer this setup because it makes it very
easy to switch implementations (obviously)! Want to send email with
sendgrid? Write an implementation. Want to send push notifications
over snail mail? Write an implementation and hire a post worker.

## Abstract Post is Abstract

This post does not cover a lot of concrete details because it cannot.
These classes represent an application. Every application is
different. There is no one size fits all approach. Hell, even if there
was we should not agree on how to implement it. Programmers, am I
right? We would disagree about what should be encapsulated where,
which methods should be public or private, or what exactly _is_ a
service object? My point is that I cannot really give you so much
advice here. Only you know what needs to be coded.

I leave you with some code from the entities in one of my apps.
Hopefully this can make this abstract discussion a little more
concrete. Here is a model, service class, and an implementation test.

<script src="https://gist.github.com/ahawkins/db56a0c35a5d025aeb61.js"></script>

## Conclusion

We are almost at the end. The data layer needs discussing. It must
come up eventually. The [next chapter]({{< ref "persistence" >}}) goes
in-depth on how to separate data, persistence, and access using the
repository and query patterns.
