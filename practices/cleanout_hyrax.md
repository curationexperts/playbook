# Get Hyrax to a Pristine State

Sometimes we want to wipe everything out of one of our systems and get it to
a pristine state. It's best if we always follow the same documented procedure
for this so we can make consistent assumptions about what has been reset. Use good judgement and TRIPLE check that you are doing this on the system you think you're doing it on. Don't accidentally wipe out production when you meant to reset the sandbox.

### 1. Shut down apache
On the command line:
```
sudo service apache2 stop
```

### 2. Clean out fedora
  If fedora is small, you can do this in a console: 
  ```
    require 'active_fedora/cleaner'
    ActiveFedora::Cleaner.clean!
  ```
  
  If fedora is huge, go to the system where fedora is running and do this:
  
  ```
  sudo service tomcat7 stop
  cd /opt
  sudo mv fedora-data fedora-data-$TODAY
  sudo mkdir fedora-data
  sudo chown tomcat7:tomcat7 fedora-data
  sudo service tomcat7 start
  ```
  
### 2. Clean out solr
  On a rails console:
  ```
  Blacklight.default_index.connection.delete_by_query("*:*"); Blacklight.default_index.connection.commit
  ```
  
### 4. Clean out redis
  On the command line: 
  ```
  redis-cli FLUSHALL
  ```
  
### 4. Re-create the database
On the command line
```
RAILS_ENV=whatever bundle exec rake db:reset
```
NOTE: You might need to restart postgres if it won't drop the database because it says you have sessions connected. `sudo service postgresql stop` then `sudo service postgresql start`

### 5. Remove uploaded files and anything in /tmp/owned by deploy
On the command line
```
sudo rm -rf /opt/uploads/hyrax/uploaded_file/file/*
sudo find /tmp -type f -user deploy -execdir /bin/rm -- {} \;
```

### 6. Restart
Restart apache and you should be ready to go!
```
sudo service apache2 start
```
