# Host Administration

## PostgreSQL database

### Database admin user

Primary admin user for PostgreSQL is `nsaph`.

Password is stored in the 
[FAS RC vault](https://vault.rc.fas.harvard.edu/ui/vault/secrets/kv/show/rse/nsaph/pg_users)
            
Another way to access database with superuser privileges is using 
`postgres` user. It is not allowed to log in as `postgres` via password 
authentication, the only way is to use the following commands
in the host terminal:

    sudo su - postgres
    psql

or 

    psql -D nsaph2
                  
> This is not a recommended way!
                 
### Creating new database user

Follow PostgreSQL documentation fr creating database user.

It is possible to use
[grant_select](https://github.com/NSAPH-Data-Platform/nsaph-core-platform/blob/c4425b43435d1ea012b3de2299a176cb014857f3/src/sql/utils.sql#L76)
SQL procedure to grant `SELECT` (i.e. read-only) privileges to all 
NSAPH schemas.

## Superset

### Superset app admin user

Administrator password for 
[Superset application](https://nsaph-superset.rc.fas.harvard.edu/login/)
is stored in
[FAS RC vault](https://vault.rc.fas.harvard.edu/ui/vault/secrets/kv/show/rse/nsaph/superset/users)

### Superset app database user

Superset uses user `superset` to communicate with teh database. 

> Humans should not use this user! 

Database password for user superset is stored in
[FAS RC vault](https://vault.rc.fas.harvard.edu/ui/vault/secrets/kv/show/rse/nsaph/pg_users)

### Creating new Superset app user
                  
A new superset user can be created using Superset app UI.
Once you are logged in as a suer with administrative privileges to
Superset app, go to Settings menu in the top right conner
and select `List Users`. Click `+` sign at the top right, next to 
the `FILTER LIST`.

