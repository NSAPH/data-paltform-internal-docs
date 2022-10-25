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
                                                                
The command(s) to create dumps are the following:

    cd /n/holystore01/LABS/dominici_lab/Lab/backup/dump
    date +%Y-%m-%d-%H-%M >> medicare_dump.log; echo "All Medicare data" >> medicare_dump.log;  nohup pg_dump -U nsaph -h localhost  -t "cms.m*" -t "medicare*.*"  -j 16 -F d -f medicare nsaph2 2>&1 >> medicare_dump.log &
    date +%Y-%m-%d-%H-%M >> gridmet_dump.log; echo "gridMET data" >> gridmet_dump.log;  nohup pg_dump -U nsaph -h localhost  -n gridmet  -j 16 -F d -f gridmet nsaph2 2>&1 >> gridmet_dump.log &
    export log=epa_dump.log ; date +%Y-%m-%d-%H-%M >> $log; echo "EPA data" >> $log;  nohup pg_dump -U nsaph -h localhost  -n epa -n pollution_wustl  -j 16 -F d -f epa nsaph2 2>&1 >> $log &
    export log=public_dump.log ; date +%Y-%m-%d-%H-%M >> $log; echo "PUBLIC data" >> $log;  nohup pg_dump -U nsaph -h localhost  -n public  -j 16 -F d -f public nsaph2 2>&1 >> $log &


