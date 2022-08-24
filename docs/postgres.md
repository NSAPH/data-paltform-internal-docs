# PostgreSQL DBMS Installation and Configuration for NSAPH Data Platform

## Installation

PostgreSQL is installed via 
[yum package manager](https://www.redhat.com/sysadmin/how-manage-packages).
Installation is managed in the puppet host file via this 
[block of code](../puppet/hieradata/hosts/holy7c26607.yaml#L253-261).

The actual installation commands are:

    yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm ;
    yum install -y postgresql13 postgresql13-server postgresql13-contrib hll_13 
    
Customized database settings are defined in the following files:

| File | Path on the host | Content in the puppet host file |
|------|------------------|---------------------------------|
| Service `override.conf` | `/etc/systemd/system/postgresql-14.service.d/override.conf` | [block](../puppet/hieradata/hosts/holy7c26607.yaml#L125-L133)|
| Overrides for `postgresql.conf` | `/data/pgsql/14/data/postgresql.override.conf` | [block](../puppet/hieradata/hosts/holy7c26607.yaml#L154-L161); [content of local::postgres::conf](../puppet/hieradata/hosts/holy7c26607.yaml#L401-L427) |
| Overrides for `pg_hba` | `/data/pgsql/14/data/pg_hba.custom.conf` | [block](../puppet/hieradata/hosts/holy7c26607.yaml#L162-L169); [content of local::postgres::pg_hba](../puppet/hieradata/hosts/holy7c26607.yaml#L429-L433) |

## Location

Data directory: `/data/pgsql/13/data`
           

