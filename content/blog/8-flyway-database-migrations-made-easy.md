---
title: 'Flyway - Database Migrations made easy & How not to accidentally Rollback all of your migrations'
date: 2019-03-24T13:09:00.002-07:00
draft: false
aliases: [ "/2019/03/flyway-database-migrations-made-easy.html" ]

image: "images/featured-post/post8.jpg"

tags:
  - "Postgres"
  - "MySQL"
  - "Best Practices"
  - "Migration"

# post type
type: "post"
---

**Flyway** - by Boxfuse: This is a **schema migration tool** and it acts as a version control system for relational databases.  
  
If you are manually executing your SQL scripts or if your administrator is manually executing the SQL scripts, on your production or UAT environments, you need this tool to be set up in all of your environments.
  
Before we proceed:  
  
**Statutory Warning: **  
  
**Never execute the following command, be it your production or UAT environment:**

**$ flyway clean   # Do not execute this, ever!!!!**  

**Wondering what it does? It roles back whatever table migrations/changes you have done through flyway, along with their data. **

**In short, Don't ever execute this command.**  
  
Now that we are done with the warnings:

![Fig1: Away, we go!](../../images/post/8-flyway-database-migrations-made-easy/img1.gif)

## Installation:

The installation of flyway, is fairly straight forward:

```
$ wget -qO- https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/5.2.4/flyway-commandline-5.2.4-linux-x64.tar.gz | tar xvz && sudo ln -s `pwd`/flyway-5.2.4/flyway /usr/local/bin
```
Run the above command in a shell prompt. This will create a directory with a name similar to: `flyway-x.x.x/`.  
Inside this directory are many other directories of which, the two most import directories are:  

| Directory | Description |
| --- | --- |
| conf/ | Configuration for each of the databases are kept here as individual conf files |
| SQL/ | SQL migrations are kept under different directories for each of the above configurations |

## Setting up the Configuration file:

If this is your first time with flyway, We would urge you to go through the configuration file from top to bottom, the boilerplate configuration file can be comical, yet scary too. Especially, this part -  quote and quote from the default configuration:  

```
# Whether to disabled clean. (default: false)  
# This is especially useful for production environments where running clean can be quite a career-limiting move.  
flyway.cleanDisabled=false  
```

The configuration is trying to convey you not to do it. So:

![Fig2: Don't do it!](../../images/post/8-flyway-database-migrations-made-easy/img2.gif)


It's all fun until one day you accidentally do a clean.  
We can't stress this enough but, make sure that this option flyway.cleanDisabled is set to true, at all costs.  

## User creation in the Database

Before we migrate the schema changes, the database user with suitable privileges need to be created.
Make sure you have two users created in your database.  
  
1) A `normal user` which should be used at all times - doesn't have delete or drop privileges:  
  
E.g.: In MySQL:  

```
mysql> create user 'flyway'@'%' identified by 'falafly';
mysql> grant create, select, insert, update, alter on $dbname.* to 'flyway'@'%' identified by 'falafly';
mysql> grant delete on $dbname.flyway_schema_history to 'flyway'@'%' identified by 'falafly';
mysql> flush privileges;
```
  
2) And a `deleteOnlyUser` which should be used only during repair operations and delete/drop operations in a database. The reason why we have such an alternate user is to have much more clear access control over the database.  
  
E.g.: In MySQL  
  
```
mysql> create user 'flywayDEL'@'%' identified by 'falafly';
mysql> grant create, select, insert, update, alter, delete on $dbname.* to 'flyway'@'%' identified by 'falafly';
mysql> grant delete on $dbname.flyway_schema_history to 'flyway'@'%' identified by 'falafly';
mysql> flush privileges;
```

## SQL Directory Setup:

Place all the SQL files in their individual directories corresponding to each of the databases under the SQL directory inside `flyway-x.x.x`. Each of the SQL files should be named with a flyway friendly convention, as:

**V1.0\_\_some\_random\_text.sql**

A common gotcha: Make sure that the V in the filename is an uppercase.

## Configuration Setup:

Delete the default configuration file under conf and substitute it with something like the following. Once again, there will be two configurations one for the default user and another for the deleteOnlyUser as:  

1) DefaultUser Configuration:

```
flyway.url = jdbc:mysql://$hostname:$portnumber/$dbname
flyway.user = flyway
flyway.password = falafly
flyway.locations = filesystem:/home/ubuntu/flyway-5.2.4/sql/$dbname
flyway.cleanDisabled = true
flyway.baselineVersion = 1
```
  
2) DeleteUser Configuration:

```
flyway.url = jdbc:mysql://$hostname:$portnumber/$dbname
flyway.user = flywayDEL
flyway.password = falafly
flyway.locations = filesystem:/home/ubuntu/flyway-5.2.4/sql/$dbname
flyway.cleanDisabled = true
flyway.baselineVersion = 1
```  
  

## All Set for migration

We are now ready to perform the migration. There are some basic commands in the flyway for retrieving information, migration, repair, and displaying the information.

**Info**

Displays the schema versions and baseline related information from the MySQL `schema_version` table.

```
$ flyway -configFiles='flyway-x.x.x/conf/$file_name.conf' info
```
  
**Migrate**  

Migrate command scans the filesystem for available migrations. It also compares these with the completed migrations. It is the centerpiece, aiding in the migration of the SQL files.

```
$ flyway -configFiles='flyway-x.x.x/conf/$file\_name.conf' migrate
```

**Repair**  

When there's a failed migration, upon correction; the checksums need to be Realigned of the applied migrations with the ones of the available migrations.  

```$ flyway -configFiles='flyway-x.x.x/conf/$file\_name.conf' repair```

**Clean**  

**Don't even think about it. If you are still wondering, it rolls back all of your migrations. Not suitable for Production/UAT/Pre prod or anywhere else.**

```$ flyway -configFiles='flyway-x.x.x/conf/$file\_name.conf' repair```

### BONUS: Migrating to a different version of flyway or Starting afresh with a new set of SQL scripts:

Let's say, our database grows in size, and there comes a scenario when the old migrations need to be archived. In that case, the following maintenance needs to be done.

* Step1: Drop the old schema history and versions.

In Mysql:

```
mysql> drop table flyway_schema_history;  
mysql> drop table schema_version;
```
  
* Step2: Alter your configuration file to locate the recent SQL files and set the baseline to a different version number.

* Step3: Baseline the database with the mentioned version.

```
$ flyway -configFiles='flyway-x.x.x/conf/$file\_name.conf' baseline
```

This will cause migrate to ignore all migrations up to and including that particular version.

**Remember; Always set your flyway.cleanDisabled as True.**