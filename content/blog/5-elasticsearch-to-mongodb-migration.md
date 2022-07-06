---
title: 'Elasticsearch to MongoDB Migration - MongoES'
date: 2018-02-11T03:05:00.000-08:00
draft: false
aliases: [ "/2018/02/elasticsearch-to-mongodb-migration.html" ]

image: "https://datawrangler.mo.cloudinary.net/images/featured-post/post5.jpg"
# meta description
description: "Informal article covering the introduction to an Elasticsearch to MongoDB migration tool known as MongoES, from the house of Datawrangler"


categories:
  - "Informal"
tags:
  - "Migration Tool"
  - "ETL"
  - "Python"
  - "MongoDB"
  - "Elasticsearch"

# post type
type: "post"

---

The following are some of the situations where the developers simply **love to hate**!

*   **The one-last-thing syndrome -** This reminds me of the following quote:

>   The first 90 percent of the code accounts for the first 90 percent of the development time. The remaining 10 percent of the code accounts for the other 90 percent of the development time.
— *Tom Cargill, Bell Labs, from the book 'Programming Pearls'*

*   **Declaring certain undocumented features to be as bugs** - Seriously, this creates traumas for the developers.
*   **Interruptions during coding** - Here's an idea. Try talking to developers while they code; chances are, they have just about <10% of your attention.

We get used to some situations. But, there are some we don't.

![Fig1: It's just a bad day](https://datawrangler.mo.cloudinary.net/images/post/5-elasticsearch-to-mongodb-migration/img1.gif)

*   **DISCONNECTION FROM THE SERVER DUE TO BAD INTERNET DURING A MIGRATION** \- Ouch!! That's gotta hurt real bad.

### Talking about ES to MongoDB Migration - How hard could that be?

**Good Side**

* JSON objects are common for both.
* Numerous tools to choose from, for migration.

**Bad Side:** 

* The Migration can be hideous, and can eat up a lot of the system resources. Be ready for a system-freeze, in case the migration tool uses a queue.

**Ugly Side:**

* Can never be resumed from the point of failure. If the connectivity goes down during the migration; the transferred collection has to be deleted and the data transfer has to be initiated once again from the beginning.  

![Fig2: No & No](https://datawrangler.mo.cloudinary.net/images/post/5-elasticsearch-to-mongodb-migration/img2.gif)

This is where MongoES can help - <a href="https://github.com/datawrangl3r/mongoes" rel="nofollow noopener" target="_blank">MongoES</a>. It is a pure python3-bred Migration tool to migrate documents from the Elasticsearch's indices to the MongoDB collections.

![Fig3: MongoES](https://datawrangler.mo.cloudinary.net/images/post/5-elasticsearch-to-mongodb-migration/img3.png)

It's robust in its native way; no queues/message brokers are involved; which means that there won't be any memory spikes or system freezes.  
  
This was achievable because *MongoES* specifically uses a tagging strategy before the migration. The tagging happens in the source Elasticsearch, which stands as a checkpoint during the migration.  

### Why a new custom id tagging, while there's an **_id** already?

Unless the documents are explicitly tagged, the `_id` fields in Elasticsearch documents are a bunch of alphanumeric strings generated to serialize the documents. These `_id` columns become unusable since queries/aggregations can not be run using them.

**MongoES - How to**  

1.  Install all the Prerequisites.
2.  Clone the repository from [https://github.com/datawrangl3r/mongoes.git](https://github.com/datawrangl3r/mongoes.git)
3.  Edit the **`mongoes.json`** file according to your requirements.

```json
{
	"EXTRACTION":
		{
			"HOST": "localhost",
			"INDEX": "lorem_ipsum",
			"DBENGINE": "elasticsearch",
			"PORT":9200
		},
	"COMMIT":
		{
			"HOST": "localhost",
			"DATABASE": "plasmodium_proteinbase",
			"COLLECTION": "mongoes",
			"DBENGINE": "mongo",
      			"USER": "",
      			"PASSWORD": "",
			"PORT":5432
		}
}
```

4.  Make sure that both the Elasticsearch and MongoDB services are up and running, and fire up the migration by keying in:

```bash
> python3 __init__.py
```

5.  Sit back and relax; for we got you covered! The migration's default value is 1000 documents per transfer.