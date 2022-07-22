# Backing up NSAPH Database

<!-- toc -->

- [Database Backup](#database-backup)
- [SQL Dump Backups](#sql-dump-backups)

<!-- tocstop -->

Please read [Database backup discussion](DatabaseBackupsDiscussion.md)
to understand why we are using 
[File-based backup](DatabaseBackupsDiscussion.md#file-system-level-backup)

## Database Backup

The primary database is using

    /n/dominici_ssd/Lab/data/pgsql/

The backup is

    /n/holystore01/LABS/dominici_lab/Lab/backup/pgsql

The following command (must be run while DBMS is down) is:

    rsync -av --progress --chown=:dominici_lab  /data/pgsql backup:/n/holystore01/LABS/dominici_lab/Lab/backup/

The host `backup` is defined in `/root/.ssh/config` by the following section:

    Host backup
     HostName   127.0.0.1
     User     mbouzinier
     IdentityFile ~/.ssh/id_rsa.mbouzinier

## SQL Dump Backups

Raw Medicare data is backed up in 

    /n/holystore01/LABS/dominici_lab/Lab/backup/dump/medicare
