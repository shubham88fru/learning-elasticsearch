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

## Analyzer
- Works on the unstructured input data so as to improve search and query results.
- Acts upon the input during indexing and querying.
- Has 3 components: Character filters -> Tokenizers -> Token filters.