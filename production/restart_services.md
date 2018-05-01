# Restarting services automatically with nagios

Production services need to be running all the time, and if they go down they
should re-start automatically without the need for human input. Here's how to 
use our nagios infrastructure to do that. This guide assumes you're using a Nagios
system (in our case, nagios.curationexperts.com) to monitor and restart a service
running on a different system, via nrpe.

This guide borrows heavily from this blog post: https://kakoma.ug/code/2014/12/nagios-event-handlers-nrpe

## On the Nagios server

### 1. Define an event handler
On nagios.curationexperts.com, edit `/usr/local/nagios/etc/objects/remotehosts.cfg` 
and add an `event_handler` to the nagios service in question. In the example 
below, `nrpe_event_handler!restart_sidekiq` will be called whenever the `check_sidekiq`
command will be called whenever nagios detects that sidekiq is down.

```
define service{
        use                     dce-service
        hostgroup_name          sidekiq
        service_description     sidekiq
        check_command           check_nrpe!check_sidekiq
        event_handler           nrpe_event_handler!restart_sidekiq
        }
```

### 2. Define a command
Add something like the following to `/usr/local/nagios/etc/objects/commands.cfg`:
```
################################################################################
# Restart sidekiq

define command{
    command_name    nrpe_event_handler
    command_line    /usr/local/nagios/libexec/event_handlers/nrpe_event_handler -s $SERVICESTATE$ -t $SERVICESTATETYPE$ -a $SERVICEATTEMPT$ -H $HOSTADDRESS$ -c $ARG1$
    }
```

### 3. Ensure nrpe_event_handler exists
On nagios.curationexperts.com, there is already a general-pupose nrpe event handler
(copied from Kakoma's blog post, above). Make sure the path is right, and that the 
file is executable. Note that the `restart_sidekiq` parameter gets passed to `nrpe_event_handler` as `ARG1`.

### 4. Restart nagios
```
sudo systemctl restart nagios
```
Make sure it came back up correctly. Nagios won't restart if there are syntax errors
in the config files. Check `/usr/local/nagios/var/nagios.log`

## On the remote (nrpe) system

### 1. Define the nrpe command
On the remote host (where nrpe is running), edit `/usr/local/nagios/etc/nrpe.cfg`
and add a line like this:
```
# Restart sidekiq if it isn't running
command[restart_sidekiq]=/usr/local/nagios/libexec/event_handlers/restart_sidekiq
```

### 2. Create the event handler
This is the file that will be executed by the nagios user if nagios detects the 
system is down. Make sure it's executable and that the nagios user has the 
permissions it needs to be able to run it. Create a file called `/usr/local/nagios/libexec/event_handlers/restart_sidekiq` (or whatever filename 
  you used in the nrpe config above). It should look something like this:

```
#!/bin/bash

#Uncomment the next two lines for debugging. Check logs in /tmp to see how execution's being done
exec 2> /tmp/nagioslog."$$"
set -x

sudo /bin/systemctl restart sidekiq
```

### 3. Restart nrpe
```
sudo systemctl restart xinetd
```

## Testing it
You should now be able to stop the monitored service. Run `tail -f /usr/local/nagios/var/log/nagios.log` on the nagios server to watch it run. You
should see nagios detect the down service, try again twice more (depending on how
  many tries you have configured -- 3 is the default). On each try it will call 
  the nrpe_event_handler but only on the third try (when the service is in a HARD down state) will it actually trigger a restart.
