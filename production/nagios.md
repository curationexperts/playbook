# Monitoring with Nagios
We monitor the systems we run with nagios. It sends out alerts if a system is down, and measures uptime. For some clients, it sends our uptime reports. For some of our contracts we have committed to maintaining a certain level of uptime, so keeping accurate statistics is important.

See also:
* [Defining a new service](define_new_service.md)
* [Restarting services automatically](restart_services.md)

## Connecting to the web UI
* The main page is `http://nagios.curationexperts.com/nagios/`. You will need nagios credentials. If you need an account, ask Bess or Mark to create one for you.

* Quick links:
  1. [All Services](http://nagios.curationexperts.com/nagios/cgi-bin/status.cgi?host=all)
  2. [Current Unhandled Service Problems](http://nagios.curationexperts.com/nagios/cgi-bin/status.cgi?host=all&type=detail&hoststatustypes=3&serviceprops=10&servicestatustypes=28)

## Receiving notifications
1. Email -- this is the default. If you want to be added to the email notifications for a service, talk to Bess or Mark.
2. On your phone -- There are several apps out there. Bess likes to use [MobiosPush](http://www.isnapp.nl/mobiospush/)
3. In a desktop client -- [NagBar](https://sites.google.com/site/nagbarapp/) is an OSX app that runs in the background and alerts you to service problems.

## Server setup
The server is `nagios.curationexperts.com`. Nagios is installed in `/usr/local/nagios`. Configuration is in `/usr/local/nagios/etc`. Please note, this directory is under local version control. If you make any changes, please identify yourself and commit your changes with a meaningful commit message. E.g.:
```
$ git config user.name "Bess Sadler"
$ git config user.email "bess@curationexperts.com"
$ git commit -a -m "Adding an entry for BarBaz University hosting contract"
```

## Emory uptime reports
We send out a weekly email report summarizing uptime for our hosted clients. Right now, our only client who gets these is Emory.

* The script to send the Emory uptime reports is `/usr/local/nagios/bin/email_pdf_reports.sh`
* The reports that have already been sent out are in `/opt/EmoryNagiosReports`

## Adding a user
1. Log in as nagios. `cd /usr/local/nagios/etc`
2. `git status` to ensure the directory is in a clean, known state
3. `htpasswd /usr/local/nagios/etc/htpasswd.users NEW_USER_NAME`. Provide a password when prompted.
4. Add the new user name to `cgi.cfg`, everywhere that user needs access.
5. `Git commit -a -m "Adding NEW_USER_NAME to nagios config"`
6. `$ sudo /etc/init.d/nagios reload`
