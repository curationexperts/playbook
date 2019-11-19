# Use Sidekiq for background jobs

We use [Sidekiq](https://github.com/mperham/sidekiq) for running background jobs.

### 1. Install Sidekiq

1. Add `gem sidekiq` to your Gemfile and run `bundle install`

### 2. Configure your Rails application to use Sidekiq

In `config/application.rb`, you can enable Sidekiq as ActiveJob's queue adapter, and get many benefits from Rails:
```ruby
  config.active_job.queue_adapter = :sidekiq
```
Samvera applications typically interact with Sidekiq using ActiveJob configured this way.

Alternatively, and less commonly, you may add a new Worker class that includes `Sidekiq::Worker` and implements the perform method:

```ruby
class HardWorker
  include Sidekiq::Worker
  def perform(name, count)
    # do something
  end
end
```

### 3. Mount the Sidekiq web UI in your routes.rb file to enable the admin panel.

This will give you a comfortable web interface for seeing and retrying failed jobs, viewing relevant error messages, and more. 
Since these tools are not for general public site users, be sure to authenticate administrators for requests to the route.

Add to `config/routes.rb`:

```ruby
  # Mount sidekiq web ui and require authentication by an admin user
  require 'sidekiq/web'
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
```

### 4. Configure your sidekiq queues
Add a file like this at `config/sidekiq.yml`:
```ruby
---
  :queues:
    - [ingest, 4]
    - [batch, 2]
    - [default, 1]

  test:
    :concurrency: 5

  development:
    :concurrency: 5

  production:
    :concurrency: 5
```
These are the queues hyrax uses by default, and you can add new ones, or tune their weighting as required. For additonal details on 
configuring queues, please see [Queues](https://github.com/mperham/sidekiq/wiki/Advanced-Options#queues) in the Sidekiq wiki.

### 5. Environments
#### a. Development
You're all set up to use Sidekiq with your application.  If you are running your application in development mode, 
you'll probably want to launch sidekiq and monitor it's activity by opening a new terminal window, changing to your
project directory and running:
```
bundle exec sidekiq
```
You can now watch Sidekiq output in that terminal window to monitor background jobs.

#### b. Test (& Continous Integration)
Running jobs asyncronously in the background can add significant complexity to your tests. 
The simplest option to reduce this complexity is to run jobs syncronously (inline) when running tests. 
You can do this by overriding the queue adapter you set in your `application.rb` with a test-specific adapter 
configured in `config/environments/test.rb`:  
```ruby
# config/enviroments/test.rb

Rails.application.configure do
  # ...
  config.active_job.queue_adapter = :test
  # ...
end
```

You can learn more about testing jobs in the "Testing Jobs" section of the 
[Testing Rails Applications](https://edgeguides.rubyonrails.org/testing.html#testing-jobs) Rails Guide, and the 
[job-specific assertions available in RSpec](https://relishapp.com/rspec/rspec-rails/docs/job-specs/job-spec).

#### c. Prodution
There are a few additonal configuration steps you'll need to take run bacground jobs in 
production.  Please see the [Sidekiq in Production](/production/sidekiq_in_production.html) 
guide for additional details and references.

### 7. Further Reading

  [Sidekiq Documentation](https://github.com/mperham/sidekiq/wiki/)
