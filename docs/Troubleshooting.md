# Data Platform Troubleshooting

<!-- toc -->

- [Restarting Services](#restarting-services)
  * [Restarting Superset](#restarting-superset)
  * [Restarting PostgreSQL](#restarting-postgresql)
  * [Restarting the Platform (including Airflow)](#restarting-the-platform-including-airflow)
- [Common Issues](#common-issues)
  * [PostgreSQL fails to restart](#postgresql-fails-to-restart)
    + [SSD disk is not mounted](#ssd-disk-is-not-mounted)
    + [Quota is full](#quota-is-full)
    + [User postgres does not have access to the disk](#user-postgres-does-not-have-access-to-the-disk)

<!-- tocstop -->

## Restarting Services

### Restarting Superset

Superset can only be restarted by a 
[user added to docker group](https://www.thegeekdiary.com/run-docker-as-a-non-root-user/). 
Alternatively, a user can have sudo privileges.

    cd /opt/superset
    source fasrc.env
    docker-compose -f docker-compose-fasrc.yml up -d

### Restarting PostgreSQL

To restart PostgreSQL, a user must have sudo privileges.

    sudo systemctl stop postgresql-13
    sudo systemctl status postgresql-13
    sudo systemctl start postgresql-13

### Restarting the Platform (including Airflow)

The platform can be only be restarted by a 
[user added to docker group](https://www.thegeekdiary.com/run-docker-as-a-non-root-user/). 
Alternatively, a user can have sudo privileges.

    cd /opt/projects/nsaph-platform-deployment
    docker-compose up -d

## Common Issues

### PostgreSQL fails to restart

#### SSD disk is not mounted

PostgreSQL database is located in teh following path:

    /data/pgsql/13/data/

Where 

    /data -> /n/dominici_ssd/Lab/data/

Therefore, if the `dominici_ssd` is not mounted, the database can 
not be brought up.

#### Quota is full

PostgreSQL must run under its own uid and gid, therefore, we 
need to check quotas for user and group `postgres` (uid=26, gid=26)

    lfs quota -g postgres -h /n/dominici_ssd

#### User postgres does not have access to the disk

User `postgres` (uid=26, gid=26) is a local user, and we use
a network file system mounted with lustre. After maintenances,
the local user might lose its access to the network drive.

Check access with teh following commands:

    sudo su - postgres
    ll /data
    ll /n/dominici_ssd/Lab

etc.
