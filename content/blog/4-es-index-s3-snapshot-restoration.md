---
title: 'Elasticsearch Index - Snapshot & Restoration in S3'
date: 2017-12-15T08:26:00.002-08:00
draft: false

aliases: [ "/2017/12/es-index-s3-snapshot-restoration.html" ]

image: "images/featured-post/post4.png"

# meta description
description: "This article explains in detail how to create a backup and restore the indexes of elasticsearch"

tags: 
- "ELK"
- "elasticsearch"
- "backup"
- "index"
- "S3"

# post type
type: "post"
---

The APIs of Elasticsearch has endpoints to create and restore the snapshots. However, the detailed documentation can be understood well with a practical example. In this article, we will be covering the basics of how the indexes can be backed up and restored from and to S3 buckets for recent and older versions of Elasticsearch. The article is divided into two sections based on the different versions of Elasticsearch:

* Elasticsearch V6 and above
* Elasticsearch V5.6 and below

## Warning

Make sure that the **elasticsearch version of the backed-up cluster/node is lesser than ( <= ) Restoring Cluster's version.** This is not a concern if the backed-up and restored clusters are the same.

## Elasticsearch V6 and above

In recent versions of Elasticsearch since V6, the security of the credentials are enhanced, and the snapshot and restore API makes use of inbuilt key stores to save the access key and secret key. If the IAM roles are attached to the service/instances with the corresponding

#### Step1: Install **S3 plugin Support**

```
> sudo bin/elasticsearch-plugin install repository-s3`  
#                                  (or)  
> sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3
```

This enables the elasticsearch instance to communicate with the AWS S3 buckets. 

#### Step2: Input the **Snapshot registration settings**

**METHOD**: PUT  
  
**URL**: `http://localhost:9200/_snapshot/logs_backup?verify=false&pretty`  
  
**PAYLOAD**:

```
{  
   "type": "s3",  
   "settings": {  
     "bucket": "WWWWWW"
   }  
 }  
```

In the URL, `logs_backup` denotes the name of the snapshot file.
  
In the payload JSON,  
        - `bucket`: `WWWWW` is where the name of the bucket is entered 

**RESPONSE**:

`{"acknowledged": "true"}`

If your compute instance/ elasticsearch service, has a policy attached to it to access the corresponding bucket, no further action is required in this step. If you have an access key and a secret key with you, the following commands need to be run and the credentials need to be provided accordingly.

```
> bin/elasticsearch-keystore add s3.client.default.access_key
> bin/elasticsearch-keystore add s3.client.default.secret_key
```

If the credentials are to be removed at some point, use the following commands:

```
> bin/elasticsearch-keystore remove s3.client.default.access_key
> bin/elasticsearch-keystore remove s3.client.default.secret_key
```

#### Step3: Cloud-Sync - list all Snapshots

**METHOD**: GET 

**URL**: `http://localhost:9200/_cat/snapshots/logs_backup?v`  

In the URL, `logs_backup` denotes the name of the snapshot file.  

The list of snapshots can be checked by executing the above command. This list provides the indices that have been backed up to the object-store. The following is the response received.

![Fig1: Listing the snapshots](../../images/post/4-es-index-s3-snapshot-restoration/img1.png)

#### Step4: Creating a Snapshot

**METHOD**: PUT  

**URL**: `http://localhost:9200/_snapshot/logs_backup/type_of_the_backup?wait_for_completion=true`  

**PAYLOAD**:

```
{  
    "indices": "logstash-2017.11.21",  
    "include_global_state": false,  
    "compress": true,  
    "encrypt": true  
}  
```  
  
In the URL, `logs_backup` denotes the name of the snapshot file and `type_of_the_backup` could be any string.
       
In the payload JSON:  
        - `indices`: Correspond to the index which is to be backed-up to an S3 bucket. In the case of multiple indices to back up under a single restoration point, the indices can be entered in the form of an array.  
        - `include_global_state`: set to 'false' just to make sure there's cross-version compatibility. **WARNING**: **If set to 'true', the index can be restored only to the ES of the source version.**  
        - `compress`: enables compression of the index meta files backed up to S3.  
        - `encrypt`: In case if extra encryption on the indices is necessary.  

**RESPONSE**:

`{"acknowledged": "true"}`
  
#### Step5: Restoring a Snapshot

**METHOD**: PUT  
  
**URL**: `http://localhost:9200/_snapshot/logs_backup/index_to_be_restored/_restore`  
  
**PAYLOAD**:

```
{  
    "ignore_unavailable": true,  
    "include_global_state": false  
}
```

In the URL, `logs_backup` denotes the name of the snapshot file and `index_to_be_restored` could be any of the indices from the `id` listed in Step3's response.

In the payload JSON:  
        - `ignore_unavailable`: It's safe to set this to true, to avoid unwanted checks.  
        - `include_global_state`: set to 'false' just to make sure there's cross-version compatibility. **WARNING**: **If set to 'true', the index can be restored only to the ES of the source version.**  
  
**RESPONSE**:

`{"acknowledged": "true"}`

## Elasticsearch V5.6 and below 

#### Step1: Install **S3 plugin Support**

```
> sudo bin/elasticsearch-plugin install repository-s3`  
#                                  (or)  
> sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3
```

This enables the elasticsearch instance to communicate with the AWS S3 buckets. 

#### Step2: Input the **Snapshot registration settings**

**METHOD**: PUT  
  
**URL**: `http://localhost:9200/_snapshot/logs_backup?verify=false&pretty`  
  
**PAYLOAD**:

```
{  
   "type": "s3",  
   "settings": {  
     "bucket": "WWWWWW",  
     "region": "us-east-1",  
     "access_key": "XXXXXX",  
     "secret_key": "YYYYYY"  
   }  
 }  
```

In the URL, `logs_backup` denotes the name of the snapshot file.
  
In the payload JSON,  
        - `bucket`: `WWWWW` is where the name of the bucket is entered  
        - `access_key` & `secret_key`: The values `XXXXXX` and `YYYYYY` is where we key in the access key and secret key for the buckets based on the `IAM policies` - If you need any help to find it, here's a link which should guide you through (https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/).  
        - `region`: The region where the bucket is hosted (choose any from http://docs.aws.amazon.com/general/latest/gr/rande.html).  

**RESPONSE**:

`{"acknowledged": "true"}`
  
#### Step3: Cloud-Sync - list all Snapshots

**METHOD**: GET 

**URL**: `http://localhost:9200/_cat/snapshots/logs_backup?v`  

In the URL, `logs_backup` denotes the name of the snapshot file.  

The list of snapshots can be checked by executing the above command. This list provides the indices that have been backed up to the object-store. The following is the response received.

![Fig1: Listing the snapshots](../../images/post/4-es-index-s3-snapshot-restoration/img1.png)

#### Step4: Creating a Snapshot

**METHOD**: PUT  

**URL**: `http://localhost:9200/_snapshot/logs_backup/type_of_the_backup?wait_for_completion=true`  

**PAYLOAD**:

```
{  
    "indices": "logstash-2017.11.21",  
    "include_global_state": false,  
    "compress": true,  
    "encrypt": true  
}  
```  
  
In the URL, `logs_backup` denotes the name of the snapshot file and `type_of_the_backup` could be any string.
       
In the payload JSON:  
        - `indices`: Correspond to the index which is to be backed-up to an S3 bucket. In the case of multiple indices to back up under a single restoration point, the indices can be entered in the form of an array.  
        - `include_global_state`: set to 'false' just to make sure there's cross-version compatibility. **WARNING**: **If set to 'true', the index can be restored only to the ES of the source version.**  
        - `compress`: enables compression of the index meta files backed up to S3.  
        - `encrypt`: In case if extra encryption on the indices is necessary.  

**RESPONSE**:

`{"acknowledged": "true"}`
  
#### Step5: Restoring a Snapshot

**METHOD**: PUT  
  
**URL**: `http://localhost:9200/_snapshot/logs_backup/index_to_be_restored/_restore`  
  
**PAYLOAD**:

```
{  
    "ignore_unavailable": true,  
    "include_global_state": false  
}
```

In the URL, `logs_backup` denotes the name of the snapshot file and `index_to_be_restored` could be any of the indices from the `id` listed in Step3's response.

In the payload JSON:  
        - `ignore_unavailable`: It's safe to set this to true, to avoid unwanted checks.  
        - `include_global_state`: set to 'false' just to make sure there's cross-version compatibility. **WARNING**: **If set to 'true', the index can be restored only to the ES of the source version.**  
  
**RESPONSE**:

`{"acknowledged": "true"}`
  
