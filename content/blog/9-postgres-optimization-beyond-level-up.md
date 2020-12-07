---
title: 'Postgres: Optimization & Beyond'
date: 2019-08-10T07:37:00.003-07:00
draft: false
aliases: [ "/2019/08/postgres-optimization-beyond-level-up.html" ]

# post thumb
image: "images/featured-post/post9.jpg"

categories:
  - "Databases"

tags:
  - "Postgres"
  - "Optimization"
  - "Databases"

# post type
type: "featured"
---

  
Postgres, one of the widely used Relational Database Management System; has been widely adopted due to its ability to handle different workloads such as web services, warehouses, etc.  

> **Fun Fact:** The name Postgres comes from it's predecessor originated from UC Berkley's Ingres Database (_INteractive GRaphics iterchangE System; meaning it's Post-INGRES_).

There are times when the performance is straightforward and in other cases when the expected performance is not met; the Database requires some tweaking in the form of structural modifications to the table, Query Tuning, Configuration improvements, etc.  
  
This article will provide some useful pointers and action plans to become a power-user in optimizing Postgres.  
  
What to do when a query is slow?  
  
![Fig1: Slow Query](../../images/post/9-postgres-optimization-beyond-level-up/img1.gif)
  
In most cases, the occurrence of a slow query is due to the absence of indexes, for those fields that are being used in the where clause of the query. That should have solved the problem, right? RIGHT?  
  
You:  

![Fig2: Are you kidding me?](../../images/post/9-postgres-optimization-beyond-level-up/img2.gif)
  
I hear you; Life ain't Fair, or _Is it?_  
  
Not all Indexes for the fields in the WHERE clause can be helpful; It all depends on the appropriate query plan prepared by the optimizer: Prepend \`EXPLAIN ANALYZE\` to the query and run it to find the query plan.  

**Pro Tip:** Use [https://explain.depesz.com/](https://explain.depesz.com/) to visualize and analyze your query plan. The color formatting gives a straight forward output to debug the reason for the slowness.  
  
The query plan itself can provide a whole lot of information about where the resources are overflowing. Given below, are a few of those keywords that you can find in the query plan and what they mean to you and the query performance.  

## Sequential Scan:

Yes, you read that right. The scan occurs sequentially; the filter runs for the whole table and returns back the rows that match the condition which can be very expensive and exhaustive. In the case of a single page / small table, Sequential scans are pretty fast.  
  
But for larger tables; To speed up the query, the sequential scan needs to be changed to an **Index Scan**. This can be done by creating indexes on the **columns that are present in the where clause**.  
  
**Index Scans / Index Only Scans:**  
  
Index Scans denote that the indexes are being properly used. Just make sure that the analyzing & vacuuming happens once in a while. This keeps all the dead tuples out of the way and allows the optimizer to choose the right index for the scan.  

## Bitmap Index Scan:

And this right here is the bummer. Bitmap Index Scans are accompanied by Bitmap Heap Scans on top. These scans occur mostly happen when one tries to retrieve multiple rows but not all, based on multiple multiple logical conditions in the where clause.  
  
Basically, it creates a bitmap out of the pages of the table, based on the condition provided (hence the Bitmap Heap Scan on top). The query can be sped up by creating a **composite index A.K.A multicolumn index**; which changes this scan to an Index Scan.  
  
**Caution:** The order of the columns in the composite index needs to be maintained the same order as that of the where clause.   

## Summarizing:  

Indexes are good; Unused Indexes are Bad;  
Having Too many Indexes is OK, as long as they are being used at some point.  
  
More RAM for the DB is Good.  
  
The VACUUM & ANALYZE of the tables is too good!!!  
ARCHIVAL of Old Data --> Being a good citizen and you are awesome!!  

#### Optimal Settings for a Postgres Engine:

For optimal performance, the following settings (requires a restart of the server) need to be made to the Postgresql conf file present in: \`_**/etc/postgresql/10/main/postgresql.conf**_\`  
  
shared Buffer - 75% of RAM  
work\_mem - 25% of RAM  
maintenance\_mem - Min: 256MB; Max:512MB  
  
Consider the scenario, where Postgres Server's has 160Gigs of RAM:  
  
shared\_buffer: 120GB  
work\_mem: 40GB  
maintenance\_mem: 256MB  

#### Steps to Optimize a query:

1) Run Explain Analyze on your Query, and if takes too long; Run Explain on your Query.  
  
2) Copy the output and paste it onto the dialogue box @ [https://explain.depesz.com/](https://explain.depesz.com/)  
  
3) Check the Stats of your query:  
  
**Index Scans / Index Only Scans** are the best and **no changes** need to be made.  
  
**Sequential Scans** can be **converted into** **Index Scans** by **creating the index for the particular column** in the where clause.  
  
**Bitmap Heap Scans** can be **converted into Index Scans** by **creating composite indexes** A.K.A multicolumn indexes, with the **same order** as that of the where clause, as:  
  
**CREATE INDEX $indexName ON $tableName ($Field1, $Field2);**  
  
Note to Self: Index & Optimize.!!  
  

![Fig3: You have got the Power!](../../images/post/9-postgres-optimization-beyond-level-up/img3.gif)