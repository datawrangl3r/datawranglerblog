---
title: 'Streaming data from Databases'
date: 2020-12-27T23:01:00.001-08:00
draft: false
aliases: [ "/2020/12/data-streams-in-java.html" ]

# post thumb
image: "images/featured-post/post13.jpg"

categories:
  - "ETL"

# meta description
description: "In this article, we will be exploring how data streaming can be implemented over JDBC connectors in Java and how we can evade the memory exhaustion issues."

tags:
  - "Java"
  - "ETL"
  - "MySQL"
  - "Databases"

# post type
type: "featured"
---

Data Streaming processes help in transferring a huge amount of data without buffering in memory. This has laid a strong foundation for the data streaming and queuing applications/services of today such as Kafka, Pulsar, etc. In this article, we will be exploring how data streaming can be implemented over JDBC connectors in Java and how we can evade the memory exhaustion issues.

<!-- Are you new to Java or do you want to refresh your Java basics? Head over to <a href="https://codegym.cc/" rel="nofollow noopener" target="_blank">https://codegym.cc/</a> and start learning today.

![Fig1: CodeGym](../../images/post/13-data-streams-in-java/img1.png) -->

### Setting up the Database

Let's set up a Database engine locally, to test our data streaming capabilities. We have set up MySQL database version `8.0.22-0` on `Ubuntu 20.04.1 LTS (Code Name: Focal)`. For Windows, you can also use WSL to install Ubuntu on your local machine. You can find the instructions to set up WSL from <a href="https://ubuntu.com/wsl" rel="nofollow noopener" target="_blank">https://ubuntu.com/wsl</a>.

```
> sudo apt-get install mysql-server
> sudo service mysql start
```

The above command installs and starts the `MySQL` service. Now that the service is up and running, let's login using the MySQL client.

```
> sudo mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.22-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  
```

For this exercise, we will be using the repository <a href="https://github.com/datawrangl3r/datastreamexample.git" rel="nofollow noopener" target="_blank">https://github.com/datawrangl3r/datastreamexample.git</a>:

```
> git clone https://github.com/datawrangl3r/datastreamexample.git
```

We now locate the cloned project's directory and source the sql file `sampledatabase.sql` inside the mysql client by running the following commands:

```
mysql> source sampledatabase.sql
Query OK, 0 rows affected, 1 warning (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

.
.
.
.

Query OK, 0 rows affected (0.00 sec)

mysql> 
```

This will create a new database known as `classicmodels`. Let's create a new user to access this database from the mysql client.

```
mysql> CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.02 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON classicmodels.* TO 'testuser'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

Now, we have a user known as `testuser`, accessible by using the password - `password` having full access to the `classicmodels` database. This database has access to the following tables.

```
mysql> show tables;
+-------------------------+
| Tables_in_classicmodels |
+-------------------------+
| customers               |
| employees               |
| offices                 |
| orderdetails            |
| orders                  |
| payments                |
| productlines            |
| products                |
+-------------------------+
8 rows in set (0.01 sec)
```

### OpenJDK runtime and JDBC Drivers

Now that our database is ready, let's install the OpenJDK runtime and the drivers to run our Java Program and access the database.

```
> sudo apt update
> sudo apt install openjdk-11-jdk
> java --version
openjdk 11.0.9 2020-10-20
OpenJDK Runtime Environment (build 11.0.9+11-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.9+11-Ubuntu-0ubuntu1.20.04, mixed mode, sharing
```

In addition to the JDK runtime, the JDBC driver can be downloaded and installed as follows: 

```
> sudo apt install libmariadb-java
```

The following entry needs to be added to `~/.bashrc` and sourced as follows:

```
export CLASSPATH=$CLASSPATH:/usr/share/java/mariadb-java-client.jar
```

```
> source ~/.bashrc
```

In the next section, we will be streaming and exporting these tables as individual CSV flat files.

### Stream Data from MySQL

<!-- If you are refreshing your basics of Java language, sign up for a free account at <a href="https://codegym.cc/" rel="nofollow noopener" target="_blank">https://codegym.cc/</a> and start learning today. With over 1200+ programming tasks with automatic verification of your solutions, Codegym can help you get up and running in no time.

![Fig2: CodeGym](../../images/post/13-data-streams-in-java/img2.png) -->

Now that we are ready, let's dive right into the code. The full code can be found at: <a href="https://github.com/datawrangl3r/datastreamexample/blob/main/connector.java" rel="nofollow noopener" target="_blank">https://github.com/datawrangl3r/datastreamexample/blob/main/connector.java</a>

The pseudocode can be elaborated as:

* Create connection object & prepare statements using java.sql.DriverManager. Set the `fetchsize` of the statement to a preferred number which denotes the number of rows to be retrieved per call.

```
Connection con = DriverManager.getConnection(url, user, password); 
Statement st = con.createStatement(java.sql.ResultSet.TYPE_FORWARD_ONLY, 
                                java.sql.ResultSet.CONCUR_READ_ONLY);
```

* Initialize an empty string ArrayList - *myTableList*

```
ArrayList <String> myTableList = new ArrayList <String> ();
```

*  Execute the query and save the contents to the initialized `ResultSet` object.

```
ResultSet rs = st.executeQuery(query);
```

* Until this object returns `False`, keep fetching and adding the table names to the arraylist defined initially.

```
String tableName = rs.getString(1);
myTableList.add(tableName);
```

* Iterate through the table names in arraylist and for each table name, perform the following actions.

  * Create another connection object & prepare statements using java.sql.DriverManager. Set the `fetchsize` of the statement to a preferred number, say 1000.

  ```
  Connection con2 = DriverManager.getConnection(url, user, password); Statement st2 = con2.createStatement(java.sql.ResultSet.TYPE_FORWARD_ONLY,
                java.sql.ResultSet.CONCUR_READ_ONLY);
   st2.setFetchSize(1000);
  ```

  * Write the headers to each file once

  * Fetch row-by-row and iterate through the columns, keep writing to the file along with the delimiter

  * Close the file when it's done

  * Repeat

As a result, we get multiple CSVs written to the script directory, each file corresponding to the entire content of the table from the MySQL database.

In the above pseudocode, the section where we define the `setFetchsize` if not defined, the code could run into an out-of-memory exception for very large datasets. This enables the JDBC driver to stream the results rather than buffering them all at once. This can also lower the fetch size down to less than 1 percent.

Another point to note is that, whenever these connections are defined, concurrent statements can not be passed with the same connection object. This puts us in a position to stream the results as fast as we can.

<!-- You could write your very own database streaming code too. Sign up for a free account at <a href="https://codegym.cc/" rel="nofollow noopener" target="_blank">https://codegym.cc/</a> and start learning Java today! -->