# Sidekiq in Production

Because sidekiq is running our background jobs, and some of these are time-
sensitive, we need to ensure sidekiq is always running, and particularly that it gets started if the system reboots. There is a good blog post
on this subject here: https://thomasroest.com/2017/03/04/properly-setting-up-redis-and-sidekiq-in-production-ubuntu-16-04.html.

In summary, sidkiq needs to be a systemctl controlled service, instead of 
a process that gets started and stopped only via a rails deploy process. Here's 
how we set that up. Most of this should be automated via ansible, but I'm documenting
it here too for future reference, because there will also be times when we have 
to do this by hand. This guide assumes Ubuntu 16.04.

### 1. Create a sidekiq system service

Create this file at `/lib/systemd/system/sidekiq.service`:

```bash
#
# systemd unit file for CentOS 7, Ubuntu 15.04
#
# Customize this file based on your bundler location, app directory, etc.
# Put this in /usr/lib/systemd/system (CentOS) or /lib/systemd/system (Ubuntu).
# Run:
#   - systemctl enable sidekiq
#   - systemctl {start,stop,restart} sidekiq
#
# This file corresponds to a single Sidekiq process.  Add multiple copies
# to run multiple processes (sidekiq-1, sidekiq-2, etc).
#
# See Inspeqtor's Systemd wiki page for more detail about Systemd:
# https://github.com/mperham/inspeqtor/wiki/Systemd
#
[Unit]
Description=sidekiq
# start us only once the network and logging subsystems are available,
# consider adding redis-server.service if Redis is local and systemd-managed.
After=syslog.target network.target

# See these pages for lots of options:
# http://0pointer.de/public/systemd-man/systemd.service.html
# http://0pointer.de/public/systemd-man/systemd.exec.html
[Service]
Type=simple
WorkingDirectory=/opt/{{ project_name }}/current
# If you use rbenv:
# ExecStart=/bin/bash -lc 'bundle exec sidekiq -e production'
# If you use the system's ruby:
ExecStart=/usr/local/bin/bundle exec sidekiq -e production -C config/sidekiq.yml -L log/sidekiq.log
User=deploy
Group=deploy
UMask=0002

# if we crash, restart
RestartSec=1
Restart=on-failure

# output goes to /var/log/syslog
StandardOutput=syslog
StandardError=syslog

# This will default to "bundler" if we don't specify it
SyslogIdentifier=sidekiq

[Install]
WantedBy=multi-user.target
```

### 2. Enable the system service
```
sudo /bin/systemctl enable sidekiq
```

### 3. Allow the deploy user to restart sidekiq without a password
Create this file at `/etc/sudoers.d/sidekiq-restart-users`:

```
deploy ALL=(ALL) NOPASSWD: /bin/systemctl start sidekiq
deploy ALL=(ALL) NOPASSWD: /bin/systemctl stop sidekiq
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart sidekiq
deploy ALL=(ALL) NOPASSWD: /bin/systemctl status sidekiq
```

### 4. Test that the deploy user can restart as expected
```
bess@qa-etd:/opt/laevigata$ sudo -u deploy bash
deploy@qa-etd:/opt/laevigata$ sudo systemctl stop sidekiq
deploy@qa-etd:/opt/laevigata$ sudo systemctl status sidekiq
● sidekiq.service - sidekiq
   Loaded: loaded (/lib/systemd/system/sidekiq.service; enabled; vendor preset: enabled)
   Active: inactive (dead)

Apr 06 10:07:22 qa-etd.library.emory.edu systemd[1]: Stopped sidekiq.
deploy@qa-etd:/opt/laevigata$ sudo systemctl start sidekiq
deploy@qa-etd:/opt/laevigata$ sudo systemctl status sidekiq
● sidekiq.service - sidekiq
   Loaded: loaded (/lib/systemd/system/sidekiq.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-04-06 10:07:34 EDT; 1s ago
 Main PID: 4098 (bundle)
    Tasks: 2
   Memory: 37.8M
      CPU: 1.531s
   CGroup: /system.slice/sidekiq.service
           └─4098 /opt/laevigata/shared/bundle/ruby/2.3.0/bin/sidekiq -e production -C config/sidekiq.yml -L log/sidekiq.log

Apr 06 10:07:34 qa-etd.library.emory.edu systemd[1]: Started sidekiq.
```

### 5. Ensure capistrano will restart sidekiq using the system service
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

### 6. Test a cap deploy and ensure everything works
