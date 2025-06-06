# Notes

- Elastic search is a non relational document database. It exposes various operations through REST APIs.

### Terminologies
- index --> table in RDBMS
- document --> row/record
- field --> column
- mapping --> schema of a table in RDBMS

### Cluster management
```http request
GET /_cluster/health    --check cluster health

###
GET /_node  --available nodes in the es cluster

###
GET /_cat/nodes?v  --stats for each node
```

### Index
```http request
### details about all avaialble indices
GET /_cat/indices?v

### deatils about indices match regex *transform*
GET /_cat/indices/*transform*?v

### create an index - note PUT not POST
PUT /products

### index with settings.
PUT /my-index
{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    }
}

### insert a doc -- Note POST
POST /books/_doc
{
  "title": "To Kill a Mockingbird",
  "author": "Harper Lee",
  "year": 1960,
  "genre": "Fiction",
  "rating": 4.9
}

### get doc by id
GET /products/_doc/<id>

### get all docs in a index
GET /products/_search

### get doc containing a query
GET /products/_search?q=kill

### delete an index
DELETE /products


### update a document with id 4
POST /books/_update/4
{
  "doc": {
    "year": 1926,
    "price": 150
  }
}

### scripted update
POST /books/_update/4
{
    "script": {
        "source": "ctx._source.price = ctx._source.price + params.value",
        "params": {
          "value": 35
        }
    }
}

### Bulk upload
POST /my-index/_bulk
{ "create": {} }
{ "name": "item1" }
{ "create": {} }
{ "name": "item2" }
{ "create": {} }
{ "name": "item3" }

### Bulk upload with id
POST /my-index/_bulk
{ "create": { "_id": 1 } }
{ "name": "item1" }
{ "create": { "_id": 2 } }
{ "name": "item2" }
{ "create": { "_id": 3 } }
{ "name": "item3" }

### Bulk create in different indexes.
POST /_bulk
{ "create": { "_index": "my-index1", "_id": 1 }}
{ "name": "item1" }
{ "create": { "_index": "my-index2", "_id": 2 }}
{ "name": "item2" }
{ "create": { "_index": "my-index3", "_id": 3 }}
{ "name": "item3" }

### Re-index - copy data from old-index to new-index
POST /_reindex
{
  "source": {
    "index": "old-index"
  },
  "dest": {
    "index": "new-index"
  }
}
```

### Analyzer
- Works on the unstructured input data so as to improve search and query results.
- Acts upon the input during indexing and querying.
- Has 3 components: Character filters -> Tokenizers -> Token filters.
```http request

### analyze api
### strip html tags etc.
POST /_analyze
{
  "text": "I work with tools like <b>Text-analyzer</b> everyday to process large datasets!!",
  "char_filter": [
    {
      "type": "html_strip"
    }
  ]
}

### map to a different string (exact matches)
POST /_analyze
{
  "text": "I work with tools like <b>Text-analyzer</b> daily to process large datasets!!",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [
        "daily => everyday",
        "- => _",
        "!! => ."
      ]
    }
  ]
}

### combine character filters.
POST /_analyze
{
  "text": "I work with tools like <b>Text-analyzer</b> daily to process large datasets!!",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [
        "daily => everyday",
        "- => _",
        "!! => ."
      ]
    },
    {
      "type": "html_strip"
    }
  ]
}

### pattern replace char filter (regex)
POST /_analyze
{
  "text": "At $100, the product is pretty expensive",
  "char_filter": [
    {
      "type": "pattern_replace",
      "pattern": "\\$(\\d+)",
      "replacement": "$1 dollars"
    }
  ]
}

### tokenizer example
POST /_analyze
{
  "text": "This is a <b>sample</b> text to see how tokens are generated.",
  "tokenizer": "standard",
  "char_filter": [
    "html_strip"
  ]
}

### Token filters
POST /_analyze
{
  "text": "This is a sample text to see how tokens are gnerated.",
  "tokenizer": "standard",
  "filter": [
    {
      "type": "uppercase"
    }
  ]
}

### custom analyzer during index creation
PUT /my-index
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1,
    "analysis": {
      "custom-analyzer": {
        "char_filter": [
          "html_strip"
        ],
        "tokenizer": "standard",
        "filter": [
          "uppercase"
        ]
      }
    }
  }
}

### Using custom analyzer
POST /my-index/_analyze
{
    "text": "<b>Hello world</b>",
    "analyzer": "cutomer-analyzer" #defaut - standard.
}
```

### Mappings
- Defines the schema of documents stored in the index (fields and their types)

```http request

### Get mapping of an index. 
GET /my-index2/_mapping

### Explicit mapping.
PUT /my-index4
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "age": { "type": "integer" },
      "email": { "type": "keyword" }
    }
  }
}
```

### Full text search
- Elasticsearch provides Query DSL (Domain specific language) to query elasticsearch (Similar to SQL for RDMS)
- QDSL is in JSON format.
- Summary:
  - Query summary -
    1. String (exact match - often used for filtering)
       -- ids, term, terms, range, exits, prefix, wildcard, regexp
    2. Flexible (partial matches, fuzziness, relevance score)
       -- match, match_phrase, multi_match, query_string
    3. Compound (combines multiple queries or clauses to form complex condn)
       -- bool, boosting, dis_max, nested

```http request
### search everything - (like  SELECT * FROM.. in SQL) 
GET /my-index/_search
{
  "query": {
    "match_all": {}
  }
}

### GET or POST both return the same result.
POST /my-index/_search
{
  "query": {
    "match_all": {}
  }
}

### Select subset based on `_id` - (like SELECT * FROM .. WHERE .. IN (...) )
GET /my-index/_search
{
  "query": {
    "ids": {
      "values": [1, 2]
    }
  }
}


### Term query (for exact match) - (like SELECT * FROM .. WHERE name = 'shubham';)
# Note that term queries are best suited for keyword type fields only because
# keyword type fields and ingored by analyzer and not modified in inverted index.
GET /my-index/_search
{
  "query": {
    "term": {
      "name": {
        "value": "item1",
        "case_insensitive": true
      }
    }
  }
}

# multiple exact match (or)
GET /my-index/_search
{
  "query": {
    "terms": {
      "name": ["item1", "item2"]
    }
  }
}


### Range queries - SELECT * FROM ... WHERE age > 5 AND age < 10
GET /my-index/_search
{
  "query": {
    "range": {
      "cost": {
        "gte": 100,
        "lte": 201
      }
    }
  }
}

### Prefix/Wildcard/Regexp - SELECT * FROM .. WHERE name LIKE 'sa%';
# Prefix search
GET /my-index/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "item"
      }
    }
  }
}

# Wildcard search
GET /my-index/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "*tem1"
      }
    }
  }
}

# Regex search
GET /my-index/_search
{
  "query": {
    "regexp": {
      "name": "it.*"
    }
  }
}


### Exists - SELECT * FROM ... WHERE .. IS NOT NULL
GET /my-index/_search
{
  "query": {
    "exists": {
      "field": "type"
    }
  }
}


### Match query - Find documents based on relevance to user's query.
GET /articles/_search
{
  "query": {
    "match": {
      "content": "spring"
    }
  }
}

# with fuzziness level
GET /articles/_search
{
  "query": {
    "match": {
      "content": {
        "query": "pring",
        "fuzziness": "1"
      }
    }
  }
}

# min prefix to be used for exact match - apply fuzziness only after this prefix length.
GET /articles/_search
{
  "query": {
    "match": {
      "content": {
        "query": "pring",
        "fuzziness": "1",
        "prefix_length": 1
      }
    }
  }
}

# Match entire phrase - only those docs that contain the entire phrase - spring boot
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "spring boot"
      }
    }
  }
}

# match spring OR season in the body field of any document.
POST /articles/_search
{
  "query": {
    "multi_match": {
      "query": "spring season",
      "fields": ["body"]
    }
  }
}

# match spring AND season in the body and title field of any document.
POST /articles/_search
{
  "query": {
    "multi_match": {
      "query": "spring season",
      "fields": ["body", "title"],
      "operator": "and"
    }
  }
}


### Bool queries
- Combines multiple queries in a flexible way.
- It has 4 clauses:
    - filter
    - must
    - must_not
    - should
- The pattern for these queries is that you'll find normal queries written within then. See e.g. below.

# Products with price less than 100 and rating greater than 4.5
POST /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "price": {
              "lte": 100
            }
          }
        },
        {
          "range": {
            "rating": {
              "gte": 4.5
            }
          }
        }
      ]
    }
  }
}

# ...AND brand must not be Nike
POST /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "price": {
              "lte": 100
            }
          }
        },
        {
          "range": {
            "rating": {
              "gte": 4.5
            }
          }
        }
      ],
      "must_not": [
        {
            "term": {
              "brand": {
                "value": "Nike"
              }
            }
        }
      ]
    }
  }
}


### E.g. - write a query - I'm looking for either shoes which are below $60 or boots with atleast 4.5 rating.
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "bool": {
            "filter": [
              {
                "range": {
                  "price": {
                    "lt": 60
                  }
                }
              }
            ],
            "must": [
              {
                "match": {
                  "name": "shoe"
                }
              }
            ]
          }
        }, 
        {
          "bool": {
            "filter": [
              {
                "range": {
                  "rating": {
                    "gte": 4.5
                  }
                }
              }
            ],
            "must": [
              {
                "match": {
                  "name": "boots"
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

### Field Selection
- Selecting specific fields from the result set -> SELECT name, age from CUSTOMER
```http request
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["name"]
}

# Paginate
# SQL LIMIT --> size
# SQL OFFSET --> from
# pagination - first page with 2 records
# size = 2
# `from` is the number of records to skip
# from = (page-1)*size

# 2 records on first apge.
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 0
}

# 2 records from second page
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "from": 2
}
```

### Aggregation
```http request
# min, max avg etc
GET /products/_search
{
  "aggs": {
    "price_max": {
      "max": {
        "field": "price"
      }
    },
    "price_min": {
      "min": {
        "field": "price"
      }
    }
  }
}

# Get price stats (min, max, avg etc) for products where brand is H&M
GET /products/_search
{
  "query": {
    "term": {
      "brand": {
        "value": "H&M"
      }
    }
  },
  "aggs": {
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}

# equivalent of group by in sql
GET /products/_search
{
  "aggs": {
    "group-by-size": {
      "terms": {
        "field": "size"
      }
    }
  }
}
```