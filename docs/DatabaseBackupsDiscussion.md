# PostgreSQL Backup Options

<!-- toc -->

- [Three backup options](#three-backup-options)
- [SQL Dumps](#sql-dumps)
- [Replication](#replication)
- [File System Level Backup](#file-system-level-backup)

<!-- tocstop -->

This document is a general discussion. See also the actual
[database backup](BackinUp.md) document.

## Three backup options

There are three different ways to back up the content of PostgreSQL DBMS, 
that are outlines in the documentation. They are:

* [SQL Dump](https://www.postgresql.org/docs/current/backup-dump.html)
* [File System Level Backup](https://www.postgresql.org/docs/current/backup-file.html)
* Replication, aka [Continuous Archiving](https://www.postgresql.org/docs/current/continuous-archiving.html)

## SQL Dumps

SQL Dump is the simplest option that is applicable to transactional databases.
NSAPH database is analytical database, i.e. the data is loaded using ETL
pipelines. In principle, restore from SQL dump is not conceptually much
different from running an ETL process. We have made some effort to optimized the
pipelines we use, therefore they will not work significantly slower than
restoring data from SQL dump. Even when using pg_dump's custom dump format we
should not be much slower than just rerunning the pipeline. Our pipelines are
parallelized, hence parallel dump does not provide significant benefit either.
SQL dumps are also not incremental by nature.

Given these considerations, **_I do not think that SQL dumps is an appropriate
general backup policy for NSAPH data platform_**. However, some of the data, with a
different ratio of the processing time to the eventual data volume can be backed
up as SQL dump. The specific dumps are described below in 
[SQL Dump Backups section](BackinUp.md#sql-dump-backups).

## Replication

The easiest way to set up continuous archiving is to establish a separate host
with a shadow server and enable 
[replication](https://www.postgresql.org/docs/current/runtime-config-replication.html). 
A shadow server creates a mirror
copy of the primary database in the background. There are several flavors of
shadow servers, in my opinion the optimal would be a 
[hot standby server](https://www.postgresql.org/docs/current/hot-standby.html).
Another
option can be a 
[warm standby server](https://www.postgresql.org/docs/current/warm-standby.html).
A really technical overview can be found
[here](https://www.postgresql.org/docs/current/different-replication-solutions.html).


Beside the official documentation referenced above, these two posts describe a
process of creating hot standby servers:

* A bit [obsolete but very detailed post](https://linuxconfig.org/how-to-create-a-hot-standby-with-postgresql). 
* A [newer, but less comprehensive post](https://www.gab.lc/articles/postgresql-12-replication/). 
 
The advantages of host standby replication are:

* The shadow server can always be used as a readonly server for queries - the main
modus operandi for the data platform. This can be useful when the primary server
is under heavy load for ingesting new data. 
* In case of a disaster, the recovery
is almost instantaneous. Basically, the shadow server is stopped, reconfigured
as a primary server, restarted and voila. All can be done in a few minutes.
* Restoration of a primary server can be done in a non-disruptive way, i.e., while
normal work continues on a former shadow server. 
* The setup is relatively simple, significantly simpler than for SQL dumps. 

The downside of replication is that it
is highly desirable to have a separate node for the shadow server. It is
technically possible to use the same host for both primary and standby servers,
but a lot of benefits and the robustness of the solution would be lost (e.g.
instant recovery).

## File System Level Backup

> Database **must** be down during file system level backup!

File system level backup is basically mirroring of the database files to another
file system. It provides robust backup option and relatively fast restore 
(within 24 to 48 hours). The main downside is that the actual **backup must happen
while the primary server is down**. We see it as a manageable constrain for NSAPH,
therefore we have selected this option.

The primary database file system resides on Lustre mounted SSD. It is being
backed up using rsync to Lustre share on holystore.

One technical obstacle that had to be solved is that no local process on the
NSAPH host can have simultaneous access to both primary and backup file system.
This is because of security concerns. The DBMS security is managed by PostgreSQL
and files are only accessible to the local root and postgres users. The secure
share on the holsytore is only accessible to a registered real user (e.g.
mbozuinier). Therefore, the actual data replication is done via local
networking, with rsync accessing the localhost using ssh logging in as a
registered user. The host configuration is updated to allow this procedure.
