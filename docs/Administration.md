# Host Administration

<!-- toc -->

- [PostgreSQL database](#postgresql-database)
  * [Database admin user](#database-admin-user)
  * [Creating new database user](#creating-new-database-user)
- [Superset](#superset)
  * [Superset app admin user](#superset-app-admin-user)
  * [Superset app database user](#superset-app-database-user)
  * [Creating new Superset app user](#creating-new-superset-app-user)
  * [Changing superset user password](#changing-superset-user-password)
    + [Changing superset user password by administrator](#changing-superset-user-password-by-administrator)
    + [Changing superset user password by the user](#changing-superset-user-password-by-the-user)
- [NGINX HTTP Server](#nginx-http-server)
  * [NGINX Virtual hosts](#nginx-virtual-hosts)
  * [NGINX HTTP Server SSL certificates](#nginx-http-server-ssl-certificates)

<!-- tocstop -->

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
                                                           
Creating new database user is described in the
[User Onboarding](Onboarding.md#create-database-user) documentation.

In essence, we follow PostgreSQL documentation for creating database user.

NSAPH users must be granted `nsaph_admin` role, i.e. they should become
members of `nsaph_admin`. This is done with the following command:

    grant nsaph_admin to username;


It is also possible to use
[grant_select](https://github.com/NSAPH-Data-Platform/nsaph-core-platform/blob/c4425b43435d1ea012b3de2299a176cb014857f3/src/sql/utils.sql#L76)
SQL procedure to grant `SELECT` (i.e. read-only) privileges to all 
NSAPH schemas:

    CALL public.grant_select('username');


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

> user `superset` is only used by Superset app for its internal purposes. It
> should nto be used for accessing the database from SQL Lab! 
> We normally use user `nsaph_superset` to access data in the database

### Database user, that Superset uses to access data in the Database

Username:  `nsaph_superset`
Password:   stored in
[FAS RC vault](https://vault.rc.fas.harvard.edu/ui/vault/secrets/kv/show/rse/nsaph/pg_users)

### Creating new Superset app user
                  
A new superset user can be created using Superset app UI.
Once you are logged in as a user with administrative privileges to
Superset app, go to `Settings` menu in the top right conner
and select `List Users`. Click `+` sign at the top right, next to 
the `FILTER LIST`.

Use any password for the user to log in for the first time and instruct
the user to change the password immediately after the first login.

Once the user has been created they should:

1. ssh to the nsaph host:

       ssh -L8088:localhost:8088  $user@nsaph.rc.fas.harvard.edu

2. Connect to Superset using the following URL: http://localhost:8088/login/ 
   * FAS RC members can use direct URL: 
       https://nsaph-superset.rc.fas.harvard.edu/login
3. Immediately [change password](#changing-superset-user-password-by-the-user).


### Changing superset user password

#### Changing superset user password by administrator

This can be done in the host terminal using superset CLI. Log in
to NSAPH host (where Superset is running) as a user either having 
[docker (i.e. member of docker group)](Troubleshooting.md#restarting-superset)
or sudo privileges. The run the following command:

    docker exec  superset_app bash -c 'superset fab reset-password --username misha --password "<new password>"'

####  Changing superset user password by the user

User should be logged in to Superset app in order to change his/her
password. Once in the app, select `Settings` in the top right corner 
and then select `Info` in the `User` section (bottom of the dropdown menu).
At the bottom left you will see `RESET MY PASSWORD` blue button.

## NGINX HTTP Server 

### NGINX Virtual hosts
                          
We use [virtual hosts](https://en.wikipedia.org/wiki/Virtual_hosting) to
enable several web applications to run on the same physical host (or VM).
At the moment, we use two applications:

* Apache Superset
* Apache Airflow

Nginx virtual hosts 
([server blocks](https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/)) 
are configured in 
puppet 
using
`nginx::nginx_vhosts` 
[block](../puppet/hieradata/hosts/holy7c26607.yaml#L323-L364)
and
`nginx::nginx_locations`
[block](../puppet/hieradata/hosts/holy7c26607.yaml#L381-L389).
The latter uses _Proxy Pass_ defined in
`local::nginx::proxy: &proxy_pass`
[block](../puppet/hieradata/hosts/holy7c26607.yaml#L366-L379).


                 
### NGINX HTTP Server SSL certificates

Once a year an SSL certificate has to be renewed.

We use InCommon multi-domain certificate.

1. Check dns for all names associated with the host. Open 
  [DNS master file](https://gitlab-int.rc.fas.harvard.edu/ops/dns/-/blob/master/address/cluster_harvard_common/common_rc_fas_harvard_edu.zone).
  and search for 'nsaph'. Note the name and all CNAMEs. 
  [Current record](https://gitlab-int.rc.fas.harvard.edu/ops/dns/-/blob/master/address/cluster_harvard_common/common_rc_fas_harvard_edu.zone#L3195-3200)
  is:

       nsaph-sandbox02 A  10.31.26.17
       
       nsaph               CNAME   holy7c26607
       nsaph-superset      CNAME   holy7c26607
       nsaph-airflow       CNAME   holy7c26607
       nsaph-docs          CNAME   holy7c26607

2. Follow [Generate CSR and Install SSL cert](https://docs-int.rc.fas.harvard.edu/generate-csr-and-ssl-cert/)
  to generate a [MULTI-DOMAIN CSR](https://docs-int.rc.fas.harvard.edu/generate-csr-and-ssl-cert/#GENERATE_A_MULTI-DOMAIN_CSR).
3. Update host [puppet file](../puppet/hieradata/hosts/holy7c26607.yaml#L438-L632).

4. Run puppet on host to bring the updates.
5. You might need to restart nginx:

        systemctl restart nginx



