---
title: 'CD-Stream: A cross database CDC Replicator Tool'
date: 2018-10-30T14:12:00.001-07:00
draft: false
aliases: [ "/2018/10/cd-streamcdc-replicator-tool-cons-on.html" ]

# post thumb
image: "images/featured-post/post7.jpg"

description: "Demonstration of CD-Stream - A cross-database change data capture tool between MySQL & Postgres Database"

categories:
  - "Informal"

tags:
  - "Change Data Capture"
  - "ETL"
  - "MySQL"
  - "Postgres"

type: "post"
---

  
Just another day at the workplace:

**5 minutes post the boot...**  

You hear everyone complain that the production database is slow. You quickly start to investigate; exploring all possible outcomes on the dashboards. 

> Could it have been the long-running slow query which you had raised a ticket for the production support to fix?.. Or Is it one of the queries run based on an un-indexed column?

![Fig1: What could it be](../../images/post/7-cd-streamcdc-replicator-tool-cons-on/img1.gif)

You hear the rants from your fellow data-analysts lamenting over their failed reports. 
You are getting a hunch. You now realize that your CPU of your Transactional Database had taken a toll by the query loads and you understand that your relational database system has gone for a toss into an eternal slumber. And all of this due to a slow running query of your ETL pipeline? Who would've thought?

You're not alone. These problems are very common in almost any organization. This is one of the main reasons why, there should be an intermediate staging layer - a data lake on which all the ETL jobs should run to populate the data warehouse. However, due to budget constraints for organizations with no access to a data lake, the only way to prevent this problem is by doing a change data capture and populating to your data warehouse directly.

### Enter CDC:

CDC is similar to database replication, except for the deletion part. All the inserts, updates, and create statements whenever issued to the databases, are usually stored in the binary logs of the databases. These binary logs are used by the additional nodes added to the database cluster or in case of restoration from failures.

![Fig2: What if I told you](../../images/post/7-cd-streamcdc-replicator-tool-cons-on/img2.jpg)

Usually, the analytical database (DWH) used in the organization is different from that of the transactional database (OLTP). The widely popular OLTP - OLAP datastore combo is: MySQL & Postgres. With that in mind, the project CD-Stream was aimed at reading from the binary logs and replicating the required tables to the destination.

It is a cross-database CDC driven replicator tool that currently supports replication between MySQL and Postgres. The tool runs queues to process the information occurring in the binary logs of the source database and replicates it across to a destination database of entirely different engines.

![Fig3: CD-Stream](../../images/post/7-cd-streamcdc-replicator-tool-cons-on/img3.png)

Post the setup, as given in the project page: <a href="https://github.com/datawrangl3r/cd-stream" rel="nofollow noopener" target="_blank">CD-Stream</a>; there's a directory called 'sample' in the project which contains some of the intensive DDL and Data Insertion scripts, for you to evaluate and exercise.