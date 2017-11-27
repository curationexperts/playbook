# Use sidekiq for background jobs

We use [sidekiq](https://github.com/mperham/sidekiq) for running background jobs.

### 1. Install sidekiq
1. Add `gem sidekiq` to your Gemfile and run `bundle install`
2. Set your application to use it, by adding this setting in `config/application.rb`:
```ruby
  config.active_job.queue_adapter = :sidekiq
```

### 2. Ensure capistrano stops and starts sidekiq on deploy
1. Add `gem 'capistrano-sidekiq', '~> 0.20.0'` to your Gemfile in the development environment and run `bundle install`
2. Add `require 'capistrano/sidekiq'` to your `Capfile`

Now when you run cap deploy it should stop and restart sidekiq

### 3. Mount sidekiq web ui:
Add to `config/routes.rb`:
```ruby
  # Mount sidekiq web ui and require authentication by an admin user
  require 'sidekiq/web'
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
```
