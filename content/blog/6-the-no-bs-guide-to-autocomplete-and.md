---
title: 'How an AutoComplete works in Elasticsearch'
date: 2018-07-03T11:53:00.003-07:00
draft: false
aliases: [ "/2018/07/the-no-bs-guide-to-autocomplete-and.html" ]

# post thumb
image: "https://datawrangler.mo.cloudinary.net/images/featured-post/post6.jpg"

# meta description
description: "An article to understand fuzzysearch and autocomplete in elasticsearch."

# taxonomies
categories:
  - "Elasticsearch"
tags:
  - "Elasticsearch"
  - "Fuzzy Search"
  - "Autocomplete"

# post type
type: "post"
---

Ever wondered how an autocomplete works in your favorite search engines? It is all about the indexing of the documents and tokenizing the keywords by applying analysis settings to them. In this article, we will be looking at how a fuzzy search and autocomplete works in elasticsearch.

![Fig1: AutoComplete](https://datawrangler.mo.cloudinary.net/images/post/6-the-no-bs-guide-to-autocomplete-and/img1.gif)

### The Basics

**Analyzer**

An *analyzer* does the analysis or *splits the indexed phrase/word into tokens/terms* upon which the search is performed with much ease.  
  
An analyzer is made up of tokenizers and filters.  
  
There are numerous analyzers in elasticsearch, by default; here, we use some of the custom analyzers tweaked to meet our requirements.

**Filter**

A filter *removes/filters keywords* from the query. Useful when we need to remove false positives from the search results based on the inputs.  
  
We will be using a stop word filter to remove the specified keywords in the search configuration from the query text.  

**Tokenizer**

The input string needs to be split, to be searched against the indexed documents. We are about to use `ngram` which splits the query text into sizeable terms.  

**Mappings**

The created analyzer needs to be mapped to a field name, for it to be efficiently used while querying.

Now that we have covered the basics, it's time to create our index.  
The first upon our index list is fuzzy search:

### Fuzzy Search

**Index Creation**

```
> curl -vX PUT http://localhost:9200/books -d @fuzzy_index.json \  
--header "Content-Type: application/json"
```

The following goes as the payload: `fuzzy_index.json`

```
{
	"mappings": {
		"books": {
			"_all": {
				"analyzer": "my_analyzer"
			},
			"properties": {
				"name": {
					"type": "string",
					"analyzer": "my_analyzer",
					"include_in_all": false
				}
			}
		}
	},
	"settings": {
		"index": {
			"analysis": {
				"analyzer": {
					"my_analyzer": {
						"tokenizer": "my_tokenizer",
						"filter": "my_filter"
					}
				},
				"filter": {
					"my_filter": {
						"type": "stop",
						"stopwords": [
							"&",
							"AND",
							"THE",
							",",
							"'"
						]
					}
				},
				"tokenizer": {
					"my_tokenizer": {
						"type": "ngram",
						"min_gram": 3,
						"max_gram": 3,
						"token_chars": [
							"letter",
							"digit"
						],
						"filter": "lowercase"
					}
				}
			}
		}
	}
}
```

And, the following books and their corresponding authors are loaded to the index.

| name	| author |
| --- | --- |
| To Kill a Mockingbird	| Harper Lee |
| When You're Ready	| J.L. Berg |
| The Book Thief | Markus Zusak |
| The Underground Railroad | Colson Whitehead |
| Pride and Prejudice | Jane Austen |
| Ready Player One | Ernest Cline |

When a fuzzy query such as the following is executed:  

```
 curl -X POST \
  http://localhost:9200/books/_search \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "query":
        {
            "match":
                {
                    "name": "ready"
                }
        }
}'
```

This query with the match keyword as "ready" returns the matched books `ready` as a keyword in the phrase as:

```
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 1.3434829,
        "hits": [
            {
                "_index": "books",
                "_type": "list",
                "_id": "AWRcM7yplopA60Y3lBSr",
                "_score": 1.3434829,
                "_source": {
                    "name": "Ready Player One",
                    "author": "Ernest Cline"
                }
            },
            {
                "_index": "books",
                "_type": "list",
                "_id": "AWRcNRwGlopA60Y3lBSs",
                "_score": 0.53484553,
                "_source": {
                    "name": "When You're Ready",
                    "author": "J.L. Berg"
                }
            }
        ]
    }
}
```

### AutoComplete

Next up, is the autocomplete. The only difference between a fuzzy search and an autocomplete is the *min_gram* and *max_gram* values.

**Index Creation**

In this case, depending on the number of characters to be auto-filled, the *min_gram* and *max_gram* values are set, as follows:

```
> curl -vX PUT http://localhost:9200/books_autocomplete -d @autocomplete.json \  
--header "Content-Type: application/json"
```

The following goes as the payload: `autocomplete.json`

```
{
	"mappings": {
		"autocomplete": {
			"_all": {
				"analyzer": "my_analyzer"
			},
			"properties": {
				"name": {
					"type": "string",
					"analyzer": "my_analyzer",
					"include_in_all": false
				}
			}
		}
	},
	"settings": {
		"index": {
			"analysis": {
				"analyzer": {
					"my_analyzer": {
						"tokenizer": "my_tokenizer",
						"filter": "my_filter"
					}
				},
				"filter": {
					"my_filter": {
						"type": "stop",
						"stopwords": [
							"&",
							"AND",
							"THE",
							",",
							"'"
						]
					}
				},
				"tokenizer": {
					"my_tokenizer": {
						"type": "ngram",
						"min_gram": 1,
						"max_gram": 30,
						"token_chars": [
							"letter",
							"digit"
						],
						"filter": "lowercase"
					}
				}
			}
		}
	}
}
```

And, the following books and their corresponding authors are loaded to the index.

| name	| author |
| --- | --- |
| To Kill a Mockingbird	| Harper Lee |
| When You're Ready	| J.L. Berg |
| The Book Thief | Markus Zusak |
| The Underground Railroad | Colson Whitehead |
| Pride and Prejudice | Jane Austen |
| Ready Player One | Ernest Cline |

When a fuzzy query such as the following is executed:  

```
 curl -X POST \
  http://localhost:9200/books_autocomplete/_search \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "query":
        {
            "match":
                {
                    "name": "ready"
                }
        }
}'
```

This query with the match keyword as "ready" returns the matched books `ready` as a keyword in the phrase as:

```
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1.3434829,
        "hits": [
            {
                "_index": "books",
                "_type": "list",
                "_id": "AWRcM7yplopA60Y3lBSr",
                "_score": 1.3434829,
                "_source": {
                    "name": "Ready Player One",
                    "author": "Ernest Cline"
                }
            }
        ]
    }
}
```

In this article, we have covered the basics of Autocomplete and Fuzzy Search. Although the syntaxes will differ, the idea remains the same across the search engines.