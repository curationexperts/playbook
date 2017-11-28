# Use Sidekiq for background jobs

We use [Sidekiq](https://github.com/mperham/sidekiq) for running background jobs.

### 1. Install Sidekiq
1. Add `gem sidekiq` to your Gemfile and run `bundle install`
2. Configure ActiveJob's queue_adapter to use it by adding this to `config/application.rb`:
```ruby
  config.active_job.queue_adapter = :sidekiq
```
Alternatively, add a Worker class that includes `Sidekiq::Worker` and implements the perform method:

```ruby
class HardWorker
  include Sidekiq::Worker
  def perform(name, count)
    # do something
  end
end
```

### 2. Ensure Capistrano stops and starts Sidekiq on deploys
1. Add `gem 'capistrano-sidekiq', '~> 0.20.0'` to your Gemfile in the development environment and run `bundle install`
2. Add `require 'capistrano/sidekiq'` to your `Capfile`

Now when you run cap deploy commands, they should stop and restart Sidekiq.

### 3. To enable the admin panel, mount the Sidekiq web UI.

This will give you a comfortable web interface for seeing and retrying failed jobs, relevant error messages, and more. Make sure only your admin users can view it.
Add to `config/routes.rb`:
```ruby
  # Mount sidekiq web ui and require authentication by an admin user
  require 'sidekiq/web'
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
```
### 4. If you had been using another background job processor, be sure to remove all of its components and configuration from your application files.

### 5. Further Reading
  [Sidekiq Documentation](https://github.com/mperham/sidekiq/wiki/)
