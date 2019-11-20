# Sidekiq in Production

Because sidekiq is running our background jobs, and some of these are time-sensitive, we need to ensure sidekiq is
always running, particularly after system reboots.

In order to start at boot time, Sidekiq needs to be managed as a system service, instead of a process that gets started
and stopped only via a rails deploy process. Here's how we set that up. Most of this should be automated via ansible,
but this guide captures the manual process that guides our ansible configuration and provides additional context and
refernce. We typically run our projects on Ubuntu based servers that use Systemd as their init system and service
manager and use Capistrano to manage our code deployments.This guide assumes Ubuntu 18.04.


### 1. Configure your application to use Sidekiq
First, add Sidekiq to your application following the steps in 
[Using Sidekiq for background jobs](/every_project/sidekiq.md). 
Once you've completed those steps and can start up Sidekiq in your development
environment, come back here and continue by configuring Capistrano (Step 2).

### 2. Configure Capistrano to be Sidekiq aware
We use the [Capistrano::Sidekiq](https://github.com/seuros/capistrano-sidekiq) gem to easily integrate Sidekiq into our
Capistrano-based deployment process. The instructions here outline the key steps we take (excerpted from
the Capistrano::Sidekiq [wiki](https://github.com/seuros/capistrano-sidekiq/wiki#install-with-rails))

Add `capistrano-sidekiq` to your Gemfile

``` ruby
gem 'capistrano-sidekiq' , group: :development
```

Load the following in your Capfile

``` ruby
require 'capistrano/sidekiq'
```

### 3. Install sidekiq as a systemd service on your server(s)
The Capistrano::Sidekiq gem provides a nice clean installer to set up Sidekiq as a Systemd managed service. The
instructions here are the key steps excerpted from Capistrano::Sidekiq [Readme - Integration with
Systemd](https://github.com/seuros/capistrano-sidekiq#integration-with-systemd)

Start by adding Sidekiq specific settings to your `config/deploy.rb` file
```ruby
# Sidekiq service defaults
set :init_system, :systemd 
set :service_unit_name, "sidekiq.service"
```
> NOTE: if not all your servers are Systemd based, include this configuration in the stage files for the servers that do run Systemd instead.

Now you need to enable Systemd to launch long-runnig services and run them as the `deploy` user at boot time. Connect to the target server
using ssh as a priveleged user and run the follwing command:  
```bash
sudo loginctl enable-linger deploy
```

Once you've set the service defaults, you can install and launch a Systemd service template by issuing the 
following command (run the command for each of your defined stages and replace _$STAGE_NAME_ correspondingly):
```bash
bundle exec cap $STAGE_NAME sidekiq:install 
```
Now Capistrano can start and stop the Sidekiq service when running deployments at the `deploy` user.

### 4. Test that the deploy user can restart Sidekiq as expected
Confirm that the `deploy` user can manage the service on the server by opening a ssh session and issing the systemctl status command: 
```bash
$ ssh deploy@curate-cd.curationexperts.com
deploy@curate-cd:~$ systemctl --user status sidekiq.service
● sidekiq.service - sidekiq for dlp-curate (cd)
   Loaded: loaded (/home/deploy/.config/systemd/user/sidekiq.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-11-19 18:34:14 EST; 5s ago
  Process: 28729 ExecStop=/bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 28733 (bundler)
   CGroup: /user.slice/user-1004.slice/user@1004.service/sidekiq.service
           └─28733 /opt/dlp-curate/shared/bundle/ruby/2.5.0/bin/sidekiq -e production

Nov 19 18:34:14 curate-cd systemd[4087]: Started sidekiq for dlp-curate (cd).
deploy@curate-cd:~$ 
```

Then confirm that you can remotely stop and start Sidekiq using capistrano (the example here is run against the stage named "cd")
```bash
Marks-MacBook-Pro-2:dlp-curate mark$ bundle exec cap cd sidekiq:stop
00:00 sidekiq:stop
      01 systemctl --user stop sidekiq.service
    ✔ 01 deploy@curate-cd.curationexperts.com 3.490s
Marks-MacBook-Pro-2:dlp-curate mark$ bundle exec cap cd sidekiq:start
00:00 sidekiq:start
      01 systemctl --user start sidekiq.service
    ✔ 01 deploy@curate-cd.curationexperts.com 1.227s
```

### 5. Tune your database pool
Ensure your database pool has enough connections to handle what sidekiq will throw at it. By default sidekiq starts 25
threads, although this is configurable. Update your `database.yml` file to match. See the [sidekiq Concurrency
documentation](https://github.com/mperham/sidekiq/wiki/Advanced-Options#concurrency) for more details.
```ruby
production:
  adapter: mysql2
  database: foo_production
  pool: 25
```
### 6. Test a cap deploy and ensure everything works
Once you've all your configuration files set, you can test your deployment. Just run the deploy command for each of your 
defined stages, replacing _$STAGE_NAME_ in the command below accordingly:
```bash
bundle exec cap $STAGE_NAME deploy 
```

Congratulations! You should now be able to visit the `/sidekiq/busy` route on your server and see a page that 
shows your running queues:

![Sidekiq queue status page](/assets/sidekiq-queues.png)



