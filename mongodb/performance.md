# MongoDB Performance

[Home](../README.md) > [MongoDB](./readme.md) > [MongoDB Performance](./new-in-4.0.md)

Units | Topics
--- | ---
[Chapter 1: Introduction](#chapter-1-introduction) | An overview of the course content.
[Chapter 2: MongoDB Indexes](#chapter-2-mongodb-indexes) | An overview of the indexes supported by MongoDB.
[Chapter 3: Index Operations](#chapter-3-index-operations) | A deep dive into how to indexes to improve performance.
[Chapter 4: Optimizing your CRUD Operations](#chapter-4-optimizing-your-crud-operations) | Different techniques to improve CRUD performance.
[Chapter 5: Performance on Clusters](#chapter-5-performance-on-clusters) | Understanding the different performance use cases for distributed systems with MongoDB.

## Chapter 1: Introduction

### Hardware Considerations & Configurations

- Memory is used for :
  - Aggregation
  - Index Traversing
  - Write Operations
  - Query Engine
  - Connections (1 MB per connection)
- CPU is attached to :
  - Storage Engine
  - Concurrency Model
  - Page Compression / Data calculation / Aggregation Framework / Map Reduce

Warning: Not all writes and read operations are non-locking operations. In this case, multiples CPUs is not necessary.

To persist data, MongoDB uses disk. We can measure performance with IOPS (Input Output Per Second).

The following table resumes the current storage capabilities :

Type | IOPS
--- | ---
7200 rpm SATA | ~ 75 - 100
15000 rpm SAS | ~ 175 - 210
SSD Intel X25-E (SLC) | ~ 5000
SSD Intel X25-M G2 (MLC) | ~ 8000
Amazon EBS | ~ 100
Amazon EBS Provisioned | ~ 2000
Amazon EBS Provisioned IOPS (SSD) | ~ 3000
Fusion IO | ~ 135 000
Violin Memory 6000 | ~ 1 000 000

Recommendation: use RAID 10 and avoid RAID 0.

## Chapter 2: MongoDB Indexes

### Introduction to Indexes

What problem Indexes try to solve ? -> `slow queries`

Imagine a `book` collection, if you want to search a specific book, the database will have to look at every single document (Collection Scan). With index, instead of see all documents in Collection, you just have to search through the ordered index first. Indexes reference every document in a collection.

`_id` field is automatically indexed.

### How Data is Stored on Disk

Remember:

- Database: logical group of collection
- Collection: operational unit that group documents together
- Index: on collection, over fields presents on documents
- Document: atomic unit of information

Start MongoDB:
`mongod --dbpath /data/db --fork --logpath /data/db/mondogb.log`

By default, almost all documents are stored in the same directory (by default /data/db, but customizable with parameter `--dbpath`). Mongo will use 1 file for collection, 1 file for index.

With option `--directoryperdb`, MongoDB will create by default 2 directories `admin` et `local`, which are the 2 default databases.

With option `--wiredTigerDirectoryForIndexes`, MongoDB will create 1 directory for collection, 1 other for index. The both inside the database directory.

These options encourage use of I/O parallelization.

### Single Field Indexes

Simplest indexes in MongoDB : `db.<collection>.createIndex({ <field>: <direction> })`

Key features :

- Keys from only one field
- Can find a single value for the indexed field
- Can find a range of values
- Can use dot notation to index fields in subdocuments
- Can be used to find several distinct values in a single query

### Sorting with indexes

`db.people.find({first_name: "James"}).sort({first_name: 1})`

Two methods for sorting :

- In memory
- Using an index

Given the following schema for the products collection:

```json
{
  "_id": ObjectId,
  "product_name": String,
  "product_id": string
}
```

And the following schema for the products collection:

```json
{ product_id: 1 }
```

Which of the following queries will use the given index to perform the sorting of the returned documents?

- `db.products.find({}).sort({ product_id: 1 })`
- `db.products.find({}).sort({ product_id: -1 })`
- `db.products.find({ product_id: '57d7a1' }).sort({ product_id: -1 })`
- `db.products.find({ product_name: 'Soap' }).sort({ product_id: 1 })`

### Querying on Compound Indexes

A compound index is an index on two or more fields. The order of these fields is very important.

If you have an index like `{lastname, firstname}`, and you query by firstname, this index is useless.

You can use index prefixes. For example, consider the following example :

```json
{ "item": 1, "location": 1, "stock": 1 }
```

The index has the following index prefixes:

- `{ item: 1 }`
- `{ item: 1, location: 1 }`

### Multikey Indexes

Consider the following document :

```json
{
  _id: ObjectId("57d7a121fa937f710a7d486d"),
  productName: "MondoDB Long Sleeve T-Shirt",
  categories: ["T-Shirts", "Clothing", "Apparel"],
  stock: [
    { size: "S", color: "red", quantity: 25 },
    { size: "S", color: "blue", quantity: 10 },
    { size: "M", color: "blue", quantity: 50 }
  ]
}
```

With Multikey Indexes, you can create this kind of index:

```powershell
db.products.createIndex({ "stock.quantity": 1 })
```

### Partial Indexes

Consider the following document (got from a collection of restaurant) :

```json
{
  "_id" : ObjectId("58a30af4b6a172ed52a8d4bf"),
  "name" : "Han Dynasty",
  "cuisine" : "Sichuan",
  "stars" : 4.4,
  "address" : {
    "street" : "90 3rd Ave",
    "city" : "New York",
    "state" : "NY",
    "zipcode" : "10003"
  }
}
```

You can create a partial index to index only restaurant with stars > 3.5 :

```powershell
db.restaurants.createIndex(
  { "address.city": 1, cuisine: 1 },
  { partialFilterExpression: { 'stars': { $gte: 3.5 } } }
)
```

This can be useful when index becomes too large to fit into memory. It is also useful with Multi Keys Indexes.
A **Sparse Index** is therefore a specific case of **Partial Index**

Example of Sparse Index :

```powershell
db.restaurants.createIndex(
  { stars: 1 },
  { sparse: true }
)
```

A Sparse Index indexes a document only if this document has a value for attribute `stars`, instead of index a null value. The same things with Partial Index :

```powershell
db.restaurants.createIndex(
  { stars: 1 },
  { partialFilterExpression: { 'stars': { $exists: true } } }
)
```

#### Partial Index Restrictions

- You cannot specify both the `partialFilterExpression` and the `sparse` options
- `_id` indexes cannot be partial indexes
- Shard key indexes cannot be partial indexes

### Text Indexes

Use text indexes involving use many index keys :

- More keys to examine
- Increase index size
- Increase time to build index
- Decrease write performance

Example with mongo shell :

```powershell
# insert 2 example documents
db.textExample.insertOne({ "statement": "MongoDB is the best" })
db.textExample.insertOne({ "statement": "MongoDB is the worst." })

# create a text index on "statement"
db.textExample.createIndex({ statement: "text" })

# Search for the phrase "MongoDB best"
db.textExample.find({ $text: { $search: "MongoDB best" } })

# Display each document with it's "textScore"
db.textExample.find({ $text: { $search : "MongoDB best" } }, { score: { $meta: "textScore" } })

# Sort the documents by their textScore so that the most relevant documents
# return first
db.textExample.find({ $text: { $search : "MongoDB best" } }, { score: { $meta: "textScore" } }).sort({ score: { $meta: "textScore" } })
```

### Collations

Several ways to define collations :

- At collection level
- In a specific query
- In an index

When you run a query, if the index collation differs from collection collation, index cannot be use.

## Chapter 3: Index Operations

### Building indexes

Two kinds of indexes :

- Foreground indexes : very fast, but block operations while index builds.
- Background indexes : slower, but don't block operations

Find all index-build:

```powershell
db.currentOp(
  {
    $or: [
      { op: "command", "query.createIndexes": { $exists: true } },
      { op: "insert", ns: /\.system\.indexes\b/ }
    ]
  }
)
```

Kill the running query using the opid (it's not really 12345)

```powershell
db.killOp(12345)
```

### Query Plans

Example of Request :

```powershell
db.restaurants.find({ "address.zipcode": { $gt: '50000' }, cuisine: 'Sushi' })
              .sort({ stars: -1 })
```

For each request, a query plan is formed, which is a series of stages : IXSCAN -> FETCH -> SORT.

Sometimes, several indexes can be match. In our example, we can have :

```json
{ _id: 1 } <- non viable
{ name: 1, cuisine: 1, stars: 1 } <- non viable
{ "address.zipcode": 1, cuisine: 1 } <- viable
{ "address.zipcode": 1, stars: 1 } <- viable
{ cuisine: 1, stars: 1 } <- viable
```

The query plans are **cached**

### Understanding Explain

`explain()` can tell us :

- Is our query using the index you expect?
- Is our query using an index to provide a sort?
- Is our query using an index to provide the projection?
- How selective is our index?
- Which part of our plan is the most expensive?

Running explain :

```powershell
db.people.find({"address.city":"Lake Meaganton"}).explain()
```

```powershell
exp = db.people.explain()
exp.find({"address.city":"Lake Meaganton"})
exp.find({"address.city":"Lake Brenda"})
```

Possible to use some parameters

```powershell
exp = db.people.explain("queryPlanner") # Default behavior
exp = db.people.explain("executionStats")
exp = db.people.explain("allPlansExecution") # Most verbose
```

### Forcing indexes with Hint

Purpose: Overriding MongoDB's default index selection with `hint()`

Example :

```powershell
db.people.find({ name: "John Doe", zipcode: { $gt: "63000" } })

# e.g. uses { name: 1, age: 1} instead of { name: 1, zipcode: 1 }
```

Use of `hint()` method :

```powershell
db.people.find(
  { name: "John Doe", zipcode: { $gt: "63000" } }
).hint(
  { name: 1, zipcode: 1 } # can also pass index name : "name_1_zipcode_1"
)
```

### Resource Allocation for Indexes

Allows us to optimize our queries, and as a side effect reduce response time.
Resource Allocation :

- Disk : to store indexes
- Memory : to operate
  
To see details about indexes :

```powershell
var stats = db.collection.stats({indexDetails:true})
stats.indexDetails # You can find here information about cache
```

When dealing with the indexes we cannot forget that

1. These data structure require resources
2. They are part of the database working set
3. We need to take them in consideration in our sizing and maintenance practices

### Basic Benchmarking

Low Level Benchmarking :

- File I/O performance
- Scheduler Performance
- Memory allocation and transfer speed
- Thread performance
- Database server performance
- Transision isolation
  
Example of tool : Sysbench / Ibench

Database Server Benchmarking :

- Data set load
- Writes per second
- Reads per second
- Balanced workloads
- Read / Write ratio

Example of tool : YCSB / TPC

Distributed Systems Benchmarking :

- Linearization
- Serialization
- Fault tolerance

Example of tool : Hibench / Jepsen

## Chapter 4: Optimizing your CRUD Operations

### Optimizing your CRUD Operations

For query :

```powershell
db.restaurants.find({ "address.zipcode": { $gt: '50000'}, cuisine: 'Sushi' })
              .sort({ stars: -1 })
```

- Index 1 : `{"address.zipcode": 1, "cuisine": 1, "stars": 1}`
- Index 2 : `{"cuisine": 1, "address.zipcode": 1, "stars": 1}`
- Index 3 : `{"cuisine": 1, "stars": 1, "address.zipcode": 1}`

Here you can find execution plans results :

Index | None | `Index 1` | `Index 2` | `Index 3`
--- | --- | --- | --- | ---
Exec Time | 386 ms | 279 ms | 90 ms | 43 ms
Docs Returned | 11,611 | 11,611 | 11,611 | 11,611
Docs Examined | 1,000,000 | 11,611 | 11,611 | 11,611
Keys Examined | 0 | 95,988 | 11,611 | 11,663
In-Memory Sort | yes | yes | yes | no

The rule for index order is : Equality / Sort / Range

### Covered Queries

```powershell
db.restaurants.find({name: { $gt: 'L' }, cuisine: 'Sushi', stars: { $gte: 4.0}}
  # Add projection
  { _id: 0, name: 1, cuisine: 1, stars: 1}
  )
```

With this index : `db.restaurants.createIndex({name: 1, cuisine: 1, stars: 1})`

You can't cover a query if :

- Any of the indexed fields are arrays
- Any of the indexed fields are embedded documents
- When run against a mongos if the index does not contain the shard key

### Insert Performance

The **write concern** describes the level of acknowledgment requested from MongoBD for write operations.
Write concern has three parameters : `{ w: <value>, j:<boolean>, wtimeout: <number> }`

- `w`: how many of the members of our replica set we're going to wait for the write
- `j`: specify whether or not we want to wait for the on disk journal
- `wtimeout`: how long we want to wait in milliseconds for the write ack. before timeout

Example of write concern : `{ w: 1, j: false, wtimeout: 5000 }`

Index Overhead :

Number of indexes | 0 | 1 | 5
--- | --- | --- | ---
Average inserts/sec | ~16,000 | ~15,000 | ~10,500
Percent loss from baseline | 0% | ~6.3% | ~34.4%

Write concern :

Write concern | w-1-j-false | w-1-j-true | w-maj-j-false | w-maj-j-true
--- | --- | --- | --- | ---
Average inserts/sec | ~27,000 | ~19,000 | ~16,000 | ~14,000
Percent loss from baseline | 0% | ~29.6% | ~40.7% | ~48.1%

### Data Types Implications

Example of documents :

```powershell
db.shapes.insert( { type: "triangle", side_length_type: "equilateral", base : 10, height: 20   } )
db.shapes.insert( { type: "triangle", side_length_type: "isosceles", base : NumberDecimal("2.8284271247461903"), side_length: 2   } )
db.shapes.insert( { type: "square", base: 1} )
db.shapes.insert( { type: "rectangle", side: 10, base: 3} )
```

```powershell
db.shapes.find({base: 2.8284271247461903})
db.shapes.find({base: "2.8284271247461903"})
db.shapes.find({base: NumberDecimal("2.8284271247461903")}) # This works
```

While sorting, MongoDB groups documents with their types in a specific order. Consider these two new documents :

```powershell
db.shapes.insert( { type: "pyramid", apex: 10, base: "3"} )
db.shapes.insert( { type: "pyramid", apex: 10, base: "14"} )
```

When sorting, you will retrieve :

```json
{ "base": 1 }
{ "base": NumberDecimal("2.8284271247461903") }
{ "base": 3 }
{ "base": 10 }
{ "base": "3" }
{ "base": "14" }
```

With `numericOrdering`, even string values will be in numeric order :

```powershell
db.shapes.createIndex({base: 1}, {collation: {locale: 'en', numericOrdering: true}})
db.shapes.find({}, {base:1, _id:0}).sort({base:1})
```

```json
{ "base": 1 }
{ "base": NumberDecimal("2.8284271247461903") }
{ "base": 3 }
{ "base": 10 }
{ "base": "14" }
{ "base": "3" }
```

### Aggregation Performance

There are two high-level categories of aggregation queries :

- `Realtime` Processing : provide data for applications, query performance is more important
- `Batch` Processing : provide data for analytics, query performance is less important

Aggregation syntax :

```powershell
db.orders.aggregate([
  { $<operator> : <predicate> },
  { $<operator> : <predicate> },
  ...
], { explain : true }) # Explain is optional
```

Imagine that we create this index in a hypothetic `orders` collection :

```json
db.orders.createIndex({ cust_id: 1})
```

```powershell
db.orders.aggregate([
  { $match : { cust_id: "287" } }, # <- Index can be used!
  { $sort : { cust_id : 1 } } # <- Index can be used!
  ...
])
```

Memory Constraints :

- Results are subject to 16MB document limit
  - Use `$limit` and `$project`
- 100MB of RAM per stage
  - Use indexes
  - Set `allowDiskUse` to true
    - Doesn't work with `$graphLookup`

## Chapter 5: Performance on Clusters

### Performance Considerations in Distributed Systems

Working with Distributed Systems :

- Consider latency
- Data is spread accross nodes
- Read implications
- Write implications

Shard **cluster** -> shard **nodes** :

- Sharding is an horizontal scaling oslution
- Have we reached the limits of our vertical scaling ?
- You need to understand how you data grows and how your data is accessed
- Sharding works by defining key based ranges - our shard key
- It's important to get a good shard key

Two kinds of read : `Routed Queries` and `Scatter Gather` (ask all nodes)

### Increasing Write Performance with Sharding

Two kinds of scaling :

- `Vertical`: server has too few resources, you have to buy more resources.
- `Horizontal`: increase the total number of servers.

`sh.shardCollection('m201.people', { last_name: 1} )`

While performing Bulk Writes, you have the choice between `ordered` and `unordered`. With ordered mode, you have to wait the end of an operation to begin the next. With unordered, all inserts will execute in parallel.

### Reading from Secondaries

```powershell
db.people.find().readPref("primary") # default option
db.people.find().readPref("secondary")
db.people.find().readPref("secondaryPreferred")
db.people.find().readPref("nearest") # lowest network latency
```

Warning: While reading from secondaries, it's not garantuee that you read the latest state of your data, because write operations begin by primary. (eventual consistency)

Reads from secondaries is a great option for : **Analytics / reporting**

Reads from secondaries is a bad idea when :

- In general
- Providing extra capacity for reads
- Reading for shards

### Replica Sets with Differing Indexes

Configure replica sets and test an index :

```powershell
var conf = {
  "_id": "M201",
  "members": [
    { "_id": 0, "host": "127.0.0.1:27000" },
    { "_id": 1, "host": "127.0.0.1:27001" },
    { "_id": 2, "host": "127.0.0.1:27002", "priority": 0 }
  ]
};
rs.initiate(conf);

db.restaurants.createIndex({"name": 1})
db.restaurants.find({name: "Perry Street Brasserie"}).explain()

db = connect("127.0.0.1:27002/m201")
db.setSlaveOk()
db.restaurants.find({name: "Perry Street Brasserie"}).explain()
```

### Aggregation Pipeline on a Sharded Cluster

Example of aggregation which use all shards in a cluster :

```powershell
db.restaurants.aggregate([
  {
    $group: {
      _id: '$address.state',
      avgStars: { $avg: '$stars' }
    }
  }
])
```

There are some aggregation optimizations by the query optimizer, for example this aggregation :

```powershell
db.restaurants.aggregate([
  {
    $sort: { stars : -1 }
  },
  {
    $match: { cuisine: 'Sushi' }
  }
])
```

will become :

```powershell
db.restaurants.aggregate([
  {
    $match: { cuisine: 'Sushi' }
  },
  {
    $sort: { stars : -1 }
  }
])
```

to reduce number of documents to sort.