# Onboarding new users

<!-- toc -->

- [Expectations from users](#expectations-from-users)
- [Giving a user access to FASSE](#giving-a-user-access-to-fasse)
- [Creating a user in Superset](#creating-a-user-in-superset)
- [[Optionally] For advanced users](#optionally-for-advanced-users)
  * [Create database user to directly access the database](#create-database-user-to-directly-access-the-database)
  * [Explain the user how to access database](#explain-the-user-how-to-access-database)
    + [Preferred way for users to connect to teh database](#preferred-way-for-users-to-connect-to-teh-database)
    + [Alternative way to connect](#alternative-way-to-connect)
  * [Create a user in Airflow](#create-a-user-in-airflow)

<!-- tocstop -->

## Expectations from users

* Users should be familiar with ssh, they should be able to generate
    ssh keys and log in to the terminal
    * Ability to use Unix command line is not strictly required
* Users should be able to establish port forwarding via an ssh tunnel
    This documentation provides enough information about how to do it 
    on Mac or Linux, but Windows users should be able to do it themselves.
* Users are supposed to be at least familiar with SQL
* Advanced users with direct access to the database should know
    database basics and be reasonably comfortable with SQL. While the
    normal advanced user permissions should prevent them from damaging
    the database it is still very easy to run a query that will take 
    several months and will use 100% of CPU.

## Giving a user access to FASSE

1. Create a user account on FASSE
2. Provide VPN access
3. Open a ticket to be inculded in PAM access to holy7c26607.rc.fas.harvard.edu 
   (srv-holy7c26607-pam)
4. Send an approval e-mail from PI (Danielle or Francesca) to Warren Frame   
5. [Obsolted?]Ask Paul Edmon to add user to a group, so they can `ssh` to nsaph host 

## Creating a user in Superset
                               
See [Creating new Superset app user](Administration.md#creating-new-superset-app-user)

Once the user has been created they should:

1. ssh to the nsaph host:

       ssh -L8088:localhost:8088 -L8280:localhost:8280 $user@nsaph.rc.fas.harvard.edu

2. Connect to Superset using the following URL: http://localhost:8088/login/ 
   * FAS RC members can use direct URL: 
       https://nsaph-superset.rc.fas.harvard.edu/login
3. Immediately [change password](Administration.md#changing-superset-user-password-by-the-user).

## [Optionally] For advanced users
           
### Create database user to directly access the database

In any SQL tool (e.g. `psql` or DBVisualizer) execute the following commands:

    CREATE user username CREATEDB  CREATEROLE LOGIN PASSWORD '111';
    grant nsaph_admin to username;

Ask the user to change the password immediately after the first login with 
the following command:

    ALTER USER username PASSWORD '<new password>';

The code of `grant_select` procedure is 
[here](https://github.com/NSAPH-Data-Platform/nsaph-core-platform/blob/c4425b43435d1ea012b3de2299a176cb014857f3/src/sql/utils.sql#L76-L93)
                          
See also [Administration guide](Administration.md#creating-new-database-user)

### Explain the user how to access database

#### Preferred way for users to connect to teh database

Use 
[DBVisualizer](https://www.dbvis.com/) 
or a similar tool (e.g. 
[Table Plus](https://tableplus.com/)
, 
[Squirrel](https://squirrel-sql.sourceforge.io/)
, etc.). There is
also a built-in SQL tool into 
[IntelliJ IDEA](https://www.jetbrains.com/idea/)
which is Java and Python IDE.
                           
For DBVisualizer you will need a Pro license or trial.
Connect using 
[ssh tunnel](https://confluence.dbvis.com/display/UG120/Security#:~:text=our%20support%20portal.-,Using%20an%20SSH%20Tunnel,accessed%20through%20an%20SSH%20tunnel).

      
#### Alternative way to connect

Use Apache Superset SQL Lab. Below are documentation on SQL Lab. 
In my opinion, it is very much inferior to DB Visualizer, but 
it can do the trick. Here are some documentation:

* [Book](https://www.oreilly.com/library/view/apache-superset-quick/9781788992244/), 
    available online at Harvard Library 
* Shorter version in 
    [ReadTheDocs](https://apache-superset.readthedocs.io/en/0.35.2/sqllab.html)
                                            
### Create a user in Airflow
                          
Access Airflow using the following URL: http://localhost:8280/

Users, connected to FAS HPRC VPN, can connect directly to:

https://nsaph-airflow.rc.fas.harvard.edu/home

Airflow allows to perform some designated administrative tasks, 
such as giving other users read or write access to the tables. It 
also allows triggering workflows from the UI and monitor the 
high level workflow progress.

> Note: users who should have only read access to the data
> should not be given Airflow access.

Log in to Airflow, select `Security` -> `List users` and click on the
`+` sign to add another user. As the user to change their password
after they log in.

> Do not forget to make user *Active* by checking the corresponding
> checkbox!

Users should be given `User` and `Op` roles.

