# Exception Logging

In production, exceptions that occur are sent to Honeybadger and are also logged with the standard Rails log which is sent to Splunk.

Blacklight (used in Hyrax) raises a `RecordNotFound` exception any time a record is not found in Solr. This is something that is very common when the site is being indexed by search engines.

A solution for this is to return a `404` instead of `500`.

[This commit](https://github.com/curationexperts/tenejo/commit/783e3ce8d58e2ccbc5823edfd05805ffd6ac9453) has all the necessary code.

For all of our apps, you should add a route like this:

`match '*path', via: :all, to: 'pages#error_404'`

That route says that for any route that doesn't exist send
the request to a controller that will return a 404 response.

Then to handle the exception you'll need to add [Rack middleware](https://guides.rubyonrails.org/rails_on_rack.html#configuring-middleware-stack) to rescue the `RecordNotFound` exception:

```
class ExceptionMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    @app.call(env)
  rescue Blacklight::Exceptions::RecordNotFound
    # Redirect to non-existant location, which goes to the 404 page
    [301, { 'Location' => '/not-found', 'Content-Type' => 'text/html' }, ['Moved Permanently']]
  end
end
```

Instead of just ignoring the exception the request is redirected to
a location that will return a `404` because of the above route. Doing
it this way requires that you add the `404` route mentioned above.

To make additional exceptions reutrn a `404` you can copy that middleware, and
add it to the stack. This should only be done if the exception occuring
isn't really exceptional. If a user or a bot requests a document that
doesn't exist, a `404` is appropriate[^1].

[^1]: See [https://tools.ietf.org/html/rfc2616#section-10.4.5](https://tools.ietf.org/html/rfc2616#section-10.4.5)
