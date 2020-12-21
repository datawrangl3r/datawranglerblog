---
title: 'Postgres to MongoDB Migrator tool - Pg2Mongo'
date: 2017-12-10T05:17:00.000-08:00
draft: false
aliases: [ "/2017/12/postgres-to-mongo-migrator-batteries.html" ]

# post thumb
image: "images/featured-post/post3.jpg"

categories:
  - "Databases"

tags:
  - "Postgres"
  - "MongoDB"
  - "ETL"
  - "Python"
  - "Migration"

# post type
type: "post"
---

Cross-database migrations have always been tedious and traumatic. We will just end up writing hours and hours of scripts to perform a task and it is useful for a single time only, which makes us think: *"All this horsepower and no room to gallop?"*
  
![Fig1: Stuck](../../images/post/3-postgres-to-mongo-migrator-batteries/img1.jpg)

## Postgres to MongoDB

There could be multiple situations when there could be a need to migrate data from a relational to a non-relational datasource.

* There could be situations where there is a need for a database change
* Legacy system change to microservices
* Change made to reveal a high performance.

Performing the switch from relational to non-relational databases can be tedious. And `Pg2Mongo` is the solution to all the migration problems to migrate data from Postgres to MongoDB.

## [Pg2Mongo](https://github.com/datawrangl3r/pg2mongo)

![Fig2: Pg2Mongo](../../images/post/3-postgres-to-mongo-migrator-batteries/img2.png)

**Pg2Mongo** is an opensource migration tool, written on python3 which gives exclusive control over cross-database migrations. The migration from relational to non-relational MongoDB can be done very easily by the following steps.

* Make sure that you have access to your very own `Postgres` and `MongoDB` servers.
* Clone the repository and install the requirements.

```
git clone https://github.com/datawrangl3r/pg2mongo
sudo pip3 install -r pg2mongo/requirements.txt
```

* Modify pg2mongo.yml based on the server names and databases that you access
* For demonstration purposes, we will be using the sample migration script in the repository `sample_migration.sql`.

```
psql -f sample_migration.sql
```

* Now that our sample data has been loaded, the script needs to understand how the transformation should occur. This can be mentioned in the `pg2mongo.yml` as

```
EXTRACTION:
    HOST : localhost 
    USER : postgres
    PASSWORD : 
    DATABASE : fandom
COMMIT:
    HOST : localhost
    USER :
    PASSWORD :
    DATABASE : fandom
MIGRATION:
    INIT_TABLE: universes
    INIT_KEYS:
        - id as uv_id
        - universe as trax_universe
        - created_at as trax_created_at
    SKELETON : 
        - KEY1 = {}
        - KEY2 = {}
    TABLES_ORDER :
        - heroes
        - weapons
    TABLES :
        heroes :
            condition : universe_id = uv_id
                #Dictionary's key ==> Postgres Field of the corresponding schema.table1
            mapping   :
                - KEY1['hero_id'] = %s['hero_id']
                - KEY1['universe_id'] = %s['universe_id']
                - KEY1['name'] = %s['name']
                - KEY1['created_at'] = %s['created_at']
                - KEY1['weapons'] = []
        weapons :
            condition : hero_id = KEY1['hero_id']
            mapping   :
                - list:
                    - KEY1['weapons'].append({})
                    - KEY1['weapons'][-1]['weapon_name'] = %s['weapon_name']
                    - KEY1['weapons'][-1]['weapon_category'] = %s['weapon_category']
    COLLECTIONS :
        heroes : KEY1
        #Collection name <== skeleton
```

It might sound overwhelming but each component is explained in detail, as follows:

| Component | What it does |
| --- | --- |
| INIT_TABLE | Initial table from which data needs to be migrated. This could be a prime table such as a transactions table with a primary key having multiple foreign constraints to other tables of the PostgreSQL database. **FOR EACH ENTRY IN THIS TABLE, THE LINKING OF OTHER TABLES WILL HAPPEN WHILE DEFINING THE TABLES.** |
| INIT_KEYS | KEYS of the init_table (aliases can be given using 'AS') |
| SKELETON | Skeleton is an empty raw python dictionary assignment which will transform to a MongoDB document, upon migration |
| TABLES_ORDER | The order by which the TABLES section needs to be executed for each of the entry from `INIT_TABLE` | 
| TABLES | Set of PostgreSQL tables enlisted along with condition and corresponding mapping. In the case of lists inside a dictionary, a list can be mentioned. Mapping is where, the association of skeleton to the table keys is defined. The value assignments are python compatible; hence, they are defined by using '%s' and other python based variable transformation functions can be used over here |
| COLLECTIONS | This is where the push of the skeleton to the corresponding MongoDB collection takes place. |

* Now that the migration template is ready, let's invoke the migration by running:

```
$ python3 __init__.py
```

You may find that based on the specified collection and transformation details mentioned in the configuration file, the migration would have completed in the destination MongoDB.

![Fig3: Relational to Non-Relational](../../images/post/3-postgres-to-mongo-migrator-batteries/img3.jpg)