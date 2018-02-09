# Clean up temp files
Various processes that Hyrax runs (e.g., ffmpeg, imagemagick, browse_everything) leave temporary files in the /tmp directory. Over time, these build up and fill up the disk, causing errors. To prevent this, put a cron job in place to remove them on a regular basis.

Also, Blacklight saves user searches, and over time these will fill up the database. They also need to be deleted regularly.

1. Install [whenever](https://github.com/javan/whenever) by adding this to your Gemfile and running bundle install:

```ruby
  gem 'whenever', require: false
```
2. In the base of your project run:

  ```ruby
  wheneverize .
  ```
3. Edit `config/schedule.rb` and add:

```ruby
  # Delete blacklight saved searches
  every :day, at: '11:55pm' do
    rake "blacklight:delete_old_searches[1]"
  end

  # Remove files in /tmp owned by the deploy user that are older than 7 days
  every :day, at: '1:00am' do
    command "/usr/bin/find /tmp -type f -mtime +7 -user deploy -execdir /bin/rm -- {} \\;"
  end
```

4. Add to `Capfile` file:

```ruby
  # use whenever to manage cron jobs
  set :whenever_command, "bundle exec whenever"
  require "whenever/capistrano"
```

Now, when you deploy the code, it will create the relevant cron jobs for the deploy user.
