# Elasticsearch

## What is Elasticsearch

Elasticsearch is a open-source search and analytics engine. It can quickly search through massive amounts of text data.  Its primary strength is finding relevant documents based on complex queries.



## Concept

- **Document:** A json object contains business data(product description, log message, blog, ...)
- **Index:** A collection of documents that have similar characteristics. Similar to "table" in RDBMS.

- **Shard**: Elasticsearch subdivides an index into multiple pieces called shards.



## Quick Start

Prerequisites: a Linux machine with JDK installed

Download Page: [Download Elasticsearch | Elastic](https://www.elastic.co/cn/downloads/elasticsearch)

First, start a standalone Elasticsearch service:

```sh
tar -xzvf elasticsearch-7.16.0-linux-x86_64.tar.gz
cd elasticsearch-7.16.0

# Optional
vim config/elasticsearch.yml

# start background
nohup bin/elasticsearch > elasticsearch.log 2>&1 &
```



Install Kibana(Optional)

Kibana Functions:

- Data Visualization
- Dev Tools (Console Query)
- Elasticsearch Management

```
tar -xzvf kibana-7.16.0-linux-x86_64.tar.gz
cd kibana-7.16.0-linux-x86_64

# server.host: "0.0.0.0"
# elasticsearch.hosts: ["http://localhost:9200"]
vim config/kibana.yml

# Run kibana background
nohup bin/kibana > kibana.log 2>&1 &
```

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



## Index & Document APIs

**Create Index:**

PUT /{indexName}

Request: 

```json
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
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

**Delete Index:**

DELETE /{indexName}

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

**Get Doc:**

GET /{indexName}/_doc/{id}



## Search API

GET /{indexName}/_search

Support Operation:

**match** 

go through tokenizer for text, not for keyword/number/date.

```
GET INDEX_NAME/_search
{
  "query": {
    "match": {
      "FIELD_NAME": "apple"
    }
  }
}

```

**term**

suitable for non-tokenized keyword field or number field.

```
GET INDEX_NAME/_search
{
  "query": {
    "term": {
      "FIELD_NAME": "success"
    }
  }
}
```

**range**

``` 
GET INDEX_NAME/_search
{
  "query": {
    "range": {
      "FIELD_NAME": {
        "gte": 100,
        "lte": 500
      }
    }
  }
}

```

**bool**

- must
- should (effect score)
- must_not
- filter

```
GET INDEX_NAME/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "apple" } }
      ],
      "filter": [
        { "range": { "price": { "lt": 100 } } }
      ],
      "must_not": [
        { "term": { "condition": "refurbished" } }
      ]
    }
  }
}
```



**sort**

For number/date/keyword

```
"sort": [
  { "price": "asc" }
]
```





**size**
For paging.

```
GET INDEX_NAME/_search
{
  "size": 10
}

```





## Cat API





## Index Management API

