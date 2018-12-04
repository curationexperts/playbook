# Steps required to restore data from one hyrax system to another

This guide was written for moving data and state from one hyrax system to another.
It was written for the Emory ETD application, but should apply to other systems with
some changes to names etc.

### Before you begin

Make sure that ssh keys have been added to the different servers that you will need
to login to and move files around.

You'll need to be able to login as ubuntu to the demo-etd server from the fedora, solr,
and Hyrax servers. This may involve generating ssh keys and copying them to demo-etd
from each of those servers.

## 1.  Restore fedora
Note: We always want to back Fedora with postgres in our production setups. This
method ensures that whatever system was used to back Fedora on the original box,
it will go into a fedora backed by postgres.

### On original fedora box:

* `mkdir /opt/fedora_export`
* `chmod 777 /opt/fedora_export`
* `curl -X POST -d "/opt/fedora_export" "http://localhost:8080/fedora/rest/fcr:backup"`
* `rsync -azhvP /opt/fedora_export ubuntu@qa-upgrade-etd.emory.edu:/opt/fedora_export`


### On the new box:
* `curl -X POST -d "/opt/fedora_export/fedora_export" "localhost:8080/fedora/rest/fcr:restore"`

## 2. Backup and Restore Postgres
### On the old system:

* Ensure you can connect to postgres on the old box
Look in `database.yml` for the production credentials. See the article [Connect to postgres if you don't know the password](reset_postgres.md) if you still can't connect.

* Make a backup of the database with `pg_dump`:

```bash
sudo -u postgres pg_dump -U prod_etd -Fc prod_etd_db > /tmp/prod_etd_pgdump.15May
```

* Copy the `pg_dump` file to the new server (**Note** Again, you might need to set up and copy some ssh keys to enable this)

```bash
scp prod_etd_pgdump.15May ubuntu@demo-etd.curationexperts.com:/home/ubuntu
```

### On the new system:

* Ensure the database and a postgres role with the expected name and permissions exists. The dump references a `prod_etd` user so you will need to a user with that name.

```
ubuntu@demo-etd:~$ sudo -u postgres psql
postgres=# CREATE ROLE prod_etd;
postgres=# ALTER ROLE prod_etd CREATEDB;
postgres=# ALTER ROLE prod_etd LOGIN;
postgres=# ALTER ROLE prod_etd ENCRYPTED PASSWORD '***';
postgres=# CREATE DATABASE prod_etd_db;
postgres=# GRANT ALL PRIVILEGES ON DATABASE prod_etd_db TO prod_etd;
```

* Ensure the user can connect:

```
ubuntu@demo-etd:~$ sudo -u postgres psql -U prod_etd prod_etd_db
Password for user prod_etd:
psql (9.5.12)
Type "help" for help.

prod_etd=>
```

* Restore database from pg_dump file:

```
ubuntu@demo-etd:~$ sudo -u postgres pg_restore -d prod_etd_db ./prod_etd_pgdump.15May
Password:
```

* Rename the database if necessary (e.g., if the old database had a non-standard name):

```
$ sudo systemctl stop apache2
$ sudo systemctl stop sidekiq
ubuntu@demo-etd:~$ sudo -u postgres psql
postgres=# DROP DATABASE laevigata;
postgres=# ALTER DATABASE prod_etd_db RENAME TO laevigata;
## These last two might be necessary to grant permissions
## to the expected database role on the renamed database
postgres=# ALTER DATABASE prod_etd_db OWNER TO whatever;
postgres=# GRANT ALL PRIVILEGES ON DATABASE whatever to whatever;
postgres=# alter user whatever with superuser;
```



## 3.  Backup and restore solr
### On the old system:

**Note:** On the emory production server the index is called `etds`, but on the new
server we want to change the core to follow the standard DCE pattern of
calling the core after the repo name. If you follow these instructions
you won't need to worry about the name differences.

If you use the Solr backup API you won't need to start and stop the
solr servers on the prod or new servers.

#### Create a backup of the solr index using the solr API:
  ```bash
  mkdir /home/ubuntu/snapshot
  chmod 777 /home/ubuntu/snapshot
  curl 'http://localhost:8983/solr/etds/replication?command=backup&location=/home/ubuntu/snapshot&name=etds'
  ```

##### Move the backup to the new server
```bash
cd /home/ubuntu/snapshot
rsync -azhvP snapshot.etds ubuntu@demo-etd.curationexperts.com:/home/ubuntu/
```

### On the new system:

#### Restore the backup to the new system

```bash
curl 'http://localhost:8983/solr/laevigata/replication?command=restore&name=etds&location=/home/ubuntu/'
```


## 4. Backup and restore redis

* Stop redis on the new server:

```bash
sudo systemctl stop redis
```

* Copy the redis dump file to the new server:

```bash
rsync -azhvP /var/lib/redis/dump.rdb ubuntu@demo-etd.curationexperts.com:/home/ubuntu/dump.rdb
```

* Move the dump to the correct location on the new server:

```bash
cp /home/ubuntu/dump.rdb /var/lib/redis/dump.rdb
```

* Restart redis on the new server:

```bash
sudo systemctl start redis
```
## 5. Copy derivatives

Copy the derivatives from the prod server to the new server:

```bash
rsync -azhvP /opt/derivatives ubuntu@demo-etd.curationexperts.comes:/home/ubuntu
```

Move the derivatives into place on the new server:
```bash
rsync -ahvP /home/ubuntu/derivatives/ /opt/derivatives
```

Optionally, remove the /home/ubuntu/derivatives/ folder if the operation was
successful.

```bash
rm -r /home/ubuntu/derivatives/
```

## 6. (Optional) Recreate derivatives

* Run the recreate derivatives task on the new server:

```bash
RAILS_ENV=production bundle exec rake derivatives:recreate_all_etds
```
