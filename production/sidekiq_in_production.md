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
[Using Sidekiq for background jobs](https://curationexperts.github.io/playbook/every_project/sidekiq.html).
Once you've completed those steps and can start up Sidekiq in your development
environment, come back here and continue by configuring Capistrano (Step 2).

### 2. Configure Capistrano to be Sidekiq aware
We use the [Capistrano::Sidekiq](https://github.com/seuros/capistrano-sidekiq) gem to integrate Sidekiq into our
Capistrano-based deployment process. The most up-to-date guide for how to configure capistrano so it can deploy to DCE built servers is [Capify a Rails app the ansible-samvera way](https://curationexperts.github.io/ansible-samvera/capification.html)

### 3. Install sidekiq as a systemd service on your server(s)
We manage this process via ansible, in [the sidekiq role in ansible-samvera](https://github.com/curationexperts/ansible-samvera/tree/master/roles/sidekiq).

### 4. Test that the deploy user can restart sidekiq as expected
Connect to your server as the deploy user via ssh and issue the command `sudo systemctl restart sidekiq`.

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
