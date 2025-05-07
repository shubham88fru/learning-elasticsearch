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

### get an index
GET /products

### delete an index
DELETE /products

### insert a doc -- Note POST
POST /books/_doc
{
  "title": "To Kill a Mockingbird",
  "author": "Harper Lee",
  "year": 1960,
  "genre": "Fiction",
  "rating": 4.9
}
```
