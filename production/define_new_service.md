# Add a monitored service in nagios
If we are running a critical service that is not yet monitored in nagios, here's
how to add it.

## On the system where the service is running

### 1. Write a script that can test the service
Put something that looks like this in `/usr/local/nagios/libexec`. This example
is `check_sidekiq`:
```bash
#!/bin/bash

function isrunning {
        # systemctl reports sidekiq running
        RUNNING=$(systemctl status sidekiq | grep running)
        if [ "$RUNNING" ]; then
                echo "OK - sidekiq is running"
                exit 0
        else
                echo "CRITICAL - sidekiq is not running"
                exit 2
        fi
}

isrunning
```
An exit code of 0 means everything is fine. An exit code of 1 is a warning, and 
an exit code of 2 means the service is in a `CRITICAL` state. Make sure this file 
is executable by the `nagios` user and that it runs as you'd expect (i.e., make
  sure it is really detecting whether the system is down).
  
### 2. Define an nrpe command to call this script
Edit `/usr/local/nagios/etc/nrpe.cfg` and add a line like this:
```bash
# Check whether sidekiq is running
command[check_sidekiq]=/usr/local/nagios/libexec/check_sidekiq
```

### 3. Restart nrpe
```
sudo systemctl restart xinetd
```

## On the nagios server

### 1. Define a service check
Edit `/usr/local/nagios/etc/objects/remotehosts.cfg` and add a service definition:
```
define service{
        use                     dce-service
        hostgroup_name          sidekiq
        service_description     sidekiq
        check_command           check_nrpe!check_sidekiq
        event_handler           nrpe_event_handler!restart_sidekiq
        }
```
Note the `event_handler` is further described in the [Restart Services](restart_services.md) guide. This is an nrpe run command, so our command
is `check_nrpe`, with the local command passed as an argument, in this case `check_sidekiq`. 

### 2. Restart nagios
```
sudo systemctl stop nagios
pkill nagios
sudo systemctl restart nagios
```
Make sure it came back up correctly. Nagios won't restart if there are syntax errors
in the config files. Check `/usr/local/nagios/var/nagios.log`
