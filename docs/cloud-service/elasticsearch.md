# Elasticsearch

## What is Elasticsearch

Elasticsearch is a open-source search and analytics engine. It can quickly search through massive amounts of text data.  Its primary strength is finding relevant documents based on complex queries.



## Concept

- **Document:** A json object contains business data(product description, log message, blog, ...)
- **Index:** A collection of documents that have similar characteristics. Similar to "table" in RDBMS.

- **Shard**: Elasticsearch subdivides an index into multiple pieces called shards.





## Basic API

**Test Connection:**

GET /

Response: 

```json
{
    "name": "ubuntu",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "ZYjaWGqATmWXA4ndLUv5Zg",
    "version": {
        "number": "7.16.0",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "6fc81662312141fe7691d7c1c91b8658ac17aa0d",
        "build_date": "2021-12-02T15:46:35.697268109Z",
        "build_snapshot": false,
        "lucene_version": "8.10.1",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```

**Create Index:**

PUT /{indexName}

Request: 

```json
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "created_at": {
        "type": "date"
      },
      "views": {
        "type": "integer"
      }
    }
  }
}
```

Response: 

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "article"
}
```



**Delete Index:**

DELETE /{indexName}

Response:

```json
{
    "acknowledged": true
}
```



**Create Doc:** 

POST /{indexName}/_doc

Request:

```json
{
  "title": "Elasticsearch Tutorial",
  "content": "This is a tutorial about Elasticsearch",
  "tags": ["search", "database"],
  "created_at": "2024-01-01",
  "views": 100
}
```

Response:

```json
{
    "_index": "article",
    "_type": "_doc",
    "_id": "ExPPoZoBi-z4cJUvnLco",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



Get Doc:

GET /{indexName}/_doc/{id}

Response:

```json
{
    "_index": "article",
    "_type": "_doc",
    "_id": "ExPPoZoBi-z4cJUvnLco",
    "_version": 1,
    "_seq_no": 0,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "Elasticsearch Tutorial",
        "content": "This is a tutorial about Elasticsearch",
        "tags": [
            "search",
            "database"
        ],
        "created_at": "2024-01-01",
        "views": 100
    }
}
```





Search All Doc:

GET /{indexName}/_search

Request:

```json
{
    "size": 10,
    "query": {
        "match_all": {}
    }
}
```

Request:

```json
{
    "query": {
        "match": {
            "title": "Elasticsearch"
        }
    }
}
```



Response:

```json
{
    "took": 11,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "article",
                "_type": "_doc",
                "_id": "ExPPoZoBi-z4cJUvnLco",
                "_score": 1.0,
                "_source": {
                    "title": "Elasticsearch Tutorial",
                    "content": "This is a tutorial about Elasticsearch",
                    "tags": [
                        "search",
                        "database"
                    ],
                    "created_at": "2024-01-01",
                    "views": 100
                }
            }
        ]
    }
}
```

