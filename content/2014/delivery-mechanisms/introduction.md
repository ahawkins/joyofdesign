---
type: docs
bookShowToC: false
---

# Delivery Mechanisms

What is a delivery mechanism? A delivery mechanism sits between the
access medium and the domain code. Here are some examples. The
[thor](http://github.com/erikhuda/thor) gem is a delivery mechanism
for CLI applications. [Sinatra](http://github.com/sinatra/sinatra) (or
any other web framework) is a delivery mechanism for HTTP.  The
consumer on the other side of the CLI or server may be machine or
human. That doesn't make a difference. The delivery mechanism makes
the application available to users. Here are some delivery mechanism
responsibilities:

* Maintain state needed to communicate to domain objects
* Instantiate form objects and use cases
* Respond with medium appropriate representation of domain objects (HTML,
  JSON, or a table written to standard output).
* Handle domain errors in a medium appropriate way (print to standard
  error display a JavaSript `alert()`).

I write all my web delivery mechanisms with Sinatra--no exceptions.
Sinatra is so light weight and flexible--and it shows.
You can compose applications from other Sinatra applications, throw
middleware all over the place, use factories to build new Sinatra apps,
and you pretty much do whatever you want with it. It's so malleable. It
also has no major dependencies which is **extremely** important.

Sinatra is the outer boundary between the domain and outside world.
The web app only deals with HTTP (delivery mechanism concerns),
instantiating the correct classes and calling them. It takes the
result and serializes it to JSON and that's a wrap. A rack request
starts with middleware and ends at the app.

The [next chapter]({{< ref "middleware" >}}) covers
[Rack](https://rack.github.io) middlware. The examples will not make
sense if you do not understand Rack's interface. You can get up to
speed by following this [guide](http://rack-bootcamp.slashdeploy.com).
