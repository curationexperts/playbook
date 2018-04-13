# Use Sidekiq for background jobs

We use [Sidekiq](https://github.com/mperham/sidekiq) for running background jobs.

### 1. Install Sidekiq

1. Add `gem sidekiq` to your Gemfile and run `bundle install`

### 2. Configure your Rails application to use Sidekiq

In `config/application.rb`, you can enable Sidekiq as ActiveJob's queue adapter, and get many benefits from Rails:
```ruby
  config.active_job.queue_adapter = :sidekiq
```
Alternatively, add a new Worker class that includes `Sidekiq::Worker` and implements the perform method:

```ruby
class HardWorker
  include Sidekiq::Worker
  def perform(name, count)
    # do something
  end
end
```

### 3. Ensure Capistrano stops and starts Sidekiq on deploys

1. Add `gem 'capistrano-sidekiq', '~> 0.20.0'` to your Gemfile in the development environment and run `bundle install`
2. Add `require 'capistrano/sidekiq'` to your `Capfile`

### 4. Ensure capistrano will restart sidekiq using the system service
See also [Sidekiq in Production](../production/sidekiq_in_production.md)
This is documented more thoroughly in the `ansible-samvera` capification document, but the relevant bit here is you must re-define some methods from `capistrano-sidekiq`. Add this to your `deploy.rb` file:

```ruby
# We have to re-define capistrano-sidekiq's tasks to work with
# systemctl in production. Note that you must clear the previously-defined
# tasks before re-defining them.
Rake::Task["sidekiq:stop"].clear_actions
Rake::Task["sidekiq:start"].clear_actions
Rake::Task["sidekiq:restart"].clear_actions
namespace :sidekiq do
  task :stop do
    on roles(:app) do
      execute :sudo, :systemctl, :stop, :sidekiq
    end
  end
  task :start do
    on roles(:app) do
      execute :sudo, :systemctl, :start, :sidekiq
    end
  end
  task :restart do
    on roles(:app) do
      execute :sudo, :systemctl, :restart, :sidekiq
    end
  end
end
```

Now when you run cap deploy commands, they should stop and restart Sidekiq.

### 5. Mount the Sidekiq web UI in your routes.rb file to enable the admin panel.

This will give you a comfortable web interface for seeing and retrying failed jobs, relevant error messages, and more. Since these tools are not for site users, be sure to authenticate administrators for requests to the route.

Add to `config/routes.rb`:

```ruby
  # Mount sidekiq web ui and require authentication by an admin user
  require 'sidekiq/web'
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
```

### 6. Tune your database pool
Ensure your database pool has enough connections to handle what sidekiq will throw at it. By default sidekiq starts 25 threads, although this is configurable. Update your `database.yml` file to match. See the [sidekiq Concurrency documentation](https://github.com/mperham/sidekiq/wiki/Advanced-Options#concurrency) for more details.
```ruby
production:
  adapter: mysql2
  database: foo_production
  pool: 25
```

### 7. Configure your sidekiq queues
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
These are the queues hyrax uses by default, and you can add new ones, or tune their weighting as required.

### 8. If you had been using another background job processor, be sure to remove all of its components and configuration from your application files.
Especially check whether there is a `queue_name_prefix` defined in your config files, which will override the queue names defined in `sidekiq.yml`.

### 9. Further Reading

  [Sidekiq Documentation](https://github.com/mperham/sidekiq/wiki/)
