# Setting up a Hyrax instance
These are best practices for setting up a DCE developed and/or hosted instance of Hyrax. Many of these are lessons learned by experience.

## For the metadata
1. Set up a master metadata spreadsheet. Mark in blue all fields that we inherit from Hyrax. Mark in green fields that we need to create. Ensure each field has an RDF predicate.

## In the code base
1. Add the random flag to rspec test suite
1. Set up CI testing with travis
1. Specify Ruby support
   1. In `Gemfile`:
   ```ruby
     ruby '2.4.2' # or other version specification as required
   ```
   1. In `.travis.yml`:
   ```yaml
   rvm:
     - 2.4.2
   ```
1. Use sidekiq for background jobs
1. Mount sidekiq UI and ensure it's limited to users with the admin role:
  1. Add to `config/routes.rb`:
  ```ruby
  # Mount sidekiq web ui and require authentication by an admin user
  require 'sidekiq/web'
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
  ```
1. Set up honeybadger.io to track exceptions
1. Set up gemnasium to track dependencies
1. Turn on eager loading across Rails enviornments. By default Rails optimizes module loading differently in `development` and `test` than in `production`. We want to normalize this behavior to avoid surprises.
  1. Add to `config/environments/development.rb` and `config/environments/test.rb`:
   ```ruby
   config.eager_load = true
   ```
1. Set the log level down to WARN especially in production. Hyrax is really really verbose and it makes debugging difficult. In `/config/environments/production.rb`:
  ```ruby
  config.log_level = :warn

  ```
1. Set up virus checking for self deposit applications (or at least ask the question)
1. Truncate titles in notifications to 140 characters, or at least test notifications against very long titles. See Laevigata.
1. Add code version in page footer. See https://github.com/curationexperts/cypripedium/pull/44/commits/cdc444d66db78e0f3697aef0b1800ead5c2d6531

## On the servers
1. Ensure the versions of ghost script, imagemagick, fits etc work together to produce non-garbled derivatives (more to come here)
1. Use the whenever gem to schedule removal of blacklight saved searches via this rake task:
  `RAILS_ENV=production bundle exec rake blacklight:delete_old_searches[1]`
1. Change location of derivatives in production environment to `/opt/derivatives` (See https://github.com/curationexperts/epigaea/pull/416) so derivatives don't disappear between deploys
