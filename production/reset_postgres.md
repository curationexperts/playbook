# Connect to postgres if you don't know the password
You should always be able to find the master postgres password in the ansible build script for the box, but we do sometimes work with legacy systems where the postgres
password wasn't recorded. In that case, this is a workaround:

Edit the `/etc/postgresql/9.5/main/pg_hba.conf` file and replace:

```
# Database administrative login by Unix domain socket
local   all             postgres                                md5
```

with:

```
# Database administrative login by Unix domain socket
local   all             postgres                                trust
```

This will allow local users to login to postgres without a password. After
you are done working with `psql` you should change this back to `md5`.
