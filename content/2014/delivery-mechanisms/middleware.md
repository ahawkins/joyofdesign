---
type: docs
---

# Middleware

A middleware stack is a mighty abstraction. A middleware can do so
many things. I began to embrace middleware after moving away from
Rails and learning more about Rack. Rack's interface is simple.
There's only one method. I started to push more and more logic into
middleware to keep the final route handler as clean as possible. I'll
cover some examples of that later. First, I want to cover my default
stack.

The default stack covers shared use cases across many different
applications.

* `Rack::BounceFavicon` - No Favicon (`rack-contrib`)
* `Rack::PostBodyContentTypeParser` - Parse JSON bodies
  (`rack-contrib`)
* `Rack::ConditionalGet` - 302 Not Modified support (`rack`)
* `Rack::Cache` - Full HTTP caching support (Varnish preferred if
  possible) (`rack-cache`)
* `Rack::Deflator` - GZipping (`rack`)
* `Manifold::Middleware` - CORS (`manifold`)
* `Harness::RackInstrumenter` - Performance Tracking `(harness-rack)`

This satisfies the bare minimum use cases: caching, JSON parsing, CORS
(if writing browser app), and gzip handling. Each application
customizes the stack from there.

Middleware can do cool stuff. It's very handy when it contains the
right logic.

## User Authentication

Web services commonly authenticate users with a token. This
authentication strategy can happen in middleware. Now the power comes.
Since it is a middleware, it can be swapped out for something else. In
the tests swap this middleware for a fake implementation that
returns a given user. The same middleware may be used to "short
circuit" the application in development so you can develop the
frontend without having to worry about authentication. Here's the
code:

```ruby
class WebService < Sinatra::Base
  class TokenAuth
    def initialize(app)
      app = @app
    end

    def call(env)
      # Get the 'X-User-Token' header
      token = env.fetch 'HTTP_X_USER_TOKEN' do
        raise "Auth header missing!"
      end

      env['app.current_user'] = UserRepo.with_token! token

      @app.call env
    end
  end

  class FakeAuth
    def initialize(app, user)
      @app, @user = app, user
    end

    def call(env)
      env['app.current_user'] = user
      @app.call env
    end
  end

  helpers do
    def current_user
      env.fetch 'app.current_user' do
        raise "no current user"
      end
    end
  end
end
```

Voilla, completely swappable authentication strategies. The final
application is independent from how `app.current_user` is set. It just
needs it to be there.

## Client Specific Conversions

Middleware is also a great place to handle client specific things. I
worked on an ember application that had some really _interesting_ JSON
structure rules. It does not make sense to build this logic into the
application itself since they are client specific. The web service
needed to take the provided data (ember data specific) make it domain
specific, take domain specific output, then convert it into a format
ember data. Creating a middleware was beneficial because requests and
response could be passed in tests. There response and received input
could than be asserted on. It did not have to involve any other
objects. There were ~15 such middleware. They shared a common format
and eventually a super class was extracted. Here is a rough outline of
what one looked liked.

```ruby
require 'rack/request'
require 'json'

class EmberDataTodoSupport
  def initialize(app)
    @app = app
  end

  def call(env)
    request = Rack::Request.new env

    if ember? request
      # Convert stuff going in
      params['todo'] = convert_todo req

      [status, headers, body] = @app.call env

      json = JSON.parse body

      # Rack will calculate the correct value
      # incorrect values will break clients
      headers.delete 'Content-Length'

      [status, headers, convert_output(json)]
    else
      @app.call env
    end

    private
    def ember?(req)
      request.env['HTTP_X_EMBER_DATA_VESION'] == '0.13'
    end

    def convert_todo(req)
      # manipulate params here
    end

    def conver_response(json)
      # manipulate output here
    end
  end
end
```

These middleware grew to contain a lot of logic over time. That was
completely ok since they were isolated and testable.

## Performance Monitoring

A middleware is a great for performance monitoring since they can wrap
an entire request/response cycle.

```ruby
class RequestPerformance
  def initialize(app, statsd)
    @app, @statsd = app, statsd
  end

  def call(env)
    @statsd.time do
      @app.call env
    end
  end
end
```

That handy middleware puts requests through a statsd timer.
This is exactly how
[harness-rack](https://github.com/ahawkins/harness-rack) works. That
middleware is included in my default stack.

## Request Bouncers

Running applications on AWS presents an interesting problem. AWS
reuses elastic IPs, so eventually you might get one that was popular.
By happenstance, one of my company's applications is getting a lot of
traffic from a samsung domain. You often want to simply ignore these
requests. A bouncer middleware works perfectly. The bouncer takes a
block. If the block returns true then the request is denied.

```class
class RequestBouncer
  def initialize(app, bouncer)
    @app, @bouncer = app, bouncer
  end

  def call(env)
    req = Rack::Request.new env
    if bouncer.call req
      [403, { }, []]
    else
      @app.call env
    end
  end
end

class NightClub < Sinatra::Base
  use RequestBouncer do |req|
    req.user_agent =~ /masscan/
  end
end
```

This is nice when you discover weird traffic patterns. Insert at top
of stack.

## Health Checks

Load balancers (HAProxy/Elastic Load Balance) require health check
urls to see if an server process can handle requests. If an
application fails the status check it should killed so a new process
can start (hopefully fixing whatever called it to fail). This happens
in two separate middlewares. There is a checker that defines the route
and executes the check. The second catches any possible exceptions and
terminates the process. They are separate because you don't want
errors in tests to kill the process. Separating them also enables you
play around with the most effective health check in development.

```ruby
HealthCheckError = Class.new RuntimeError

class StatusCheck
  def initialize(app, check = nil)
    @app, @check = app, check
  end

  def call(env)
    if env['PATH_INFO'] == '/status'
     if @check
        begin
          result = @block.call(::Rack::Request.new(env))
          raise "health check did not return correctly" unless result
        rescue => boom
          fail HealthCheckError, boom.to_s
        end

        [200, {'Content-Type' => 'text/plain'}, ['Goliath Online!']]
      end
    else
      @app.call env
    end
  end
end

class Executioner
  def initialize(app)
    @app = app
  end

  def call(env)
    begin
      @app.call env
    rescue HealthCheckError => ex
      env['rack.errors'].write ex.to_s
      env['rack.errors'].write ex.backtrace.join("\n")
      env['rack.errors'].flush
      exit!
    end
  end
end
```

The status check middlware optionally takes a block. The block can be
used to test connections to external services (like a DB). This is
especially useful with MySQL since connections expire if not used
after a while. Constant pinging from the load balancer will keep
everything open. Here's how to use it with Sinatra.

```ruby
class App < Sinatra::Base
  # Must be before StatusCheck is inserted
  configure :staging, :production do
    use Executioner
  end

  use StatusCheck do
    Sequel.db.connected? && App.redis.connected?
  end
end
```

## Insert Middleware Chains or Other Applications

The rack interface enables a lot of fun things. The object must simply
respond to `call`. It maybe a single class or a new chain. This means
you can insert a whole Sinatra app into the middleware chain or simply
build more complex middleware chains. The ember data conversion middlewares
could be an entire application then inserted.

```ruby
class EmberDataConversionPipeline
  def initialize(app)
    stack = Rack::Buidler
    stack.use TodoConverter
    stack.use ContactConveter
    stack.use EmailConverter
    stack.run app

    @app = stack.to_app
  end

  def call(env)
    @app.call env
  end
end

class WebService < Sinatra::Base
  use EmberDataConversionPipeline
end
```

I use this pattern when I want to group a bunch of related middleware
together. I do this because it makes the resulting application's
middleware stack easier to read. Here's another real life example.
This middleware insert the harness rack instrumentation and rack's
runtime middleware. The request's time is logged to statsd and a
`X-Runtime` header is added.

```ruby
class Instrumentation
  def initialize(app, namespace = nil)
    stack = ::Rack::Builder.new
    stack.use ::Rack::Runtime
    stack.use ::Harness::RackInstrumenter, namespace
    stack.run app

    @app = stack.to_app
  end

  def call(env)
    @app.call env
  end
end
```

These examples are here to illustrate that a middleware does not have
to be a single class. You really can build up a powerful middleware
stack.

I hope these examples were helpful for you or revealed some things you
could do in your application today.

The [next chapter]({{< ref "helpers-and-error-handling" >}}) covers
helpers and error handling.
