# New Features and Tools in MongoDB 4.0

[Home](../README.md) > [MongoDB](./readme.md) > [New Features and Tools in MongoDB 4.0](./new-in-4.0.md)

## Chapter 1 : Replica Set Transactions

### Introduction to Replica Set Transactions

- `ACID Transaction` at document level (multi-document supported)
- Transactions across Cluster will be release in `Mongo 4.2`

### Transaction Considerations

- [WiredTiger Cache] -> proactively evict snapshots and abort transactions if cache pressure thresholds are reached
- Abort automatically transactions > 60 seconds -> fully rolled back
- Best practice : no more **1000 documents modified** in 1 transaction
- If error : retry the commit statement
- DDL operations (create index, drop collection) block transactions -> if a transaction comes at the same time, transaction will abort

### Write Conflicts

- If Transaction1 -> Document1, and Transaction2 -> Document1 => Transaction2 will abort with a write conflicts

### Replica Set Transactions

- 80-90% projects not concerned by multi-document transactions (40 years Relational data modeling)
- All-or-nothing execution
- Fully expressive JOINs : $lookup aggregation pipeline stage

### Abort vs Commit

- TransientTransactionError : Connection error or Write conflicts (ConnectionFailure / OperationFailure)

## Chapter 2 : Sharding Improvements

### Sharding: Balancing improvements

- MongoDB enables elastically add and remove nodes from a sharded cluster in real time
- Automatically rebalancing data across nodes **40% faster**

### Sharding: Logging of Slow Queries and Sharded Kill on mongos

- Extended debuggability in the mongos : `<timestamp> <severity> <component> [<context>] <message>`
- Sharded Kill : Ability to kill operations across shard clusters, mongos will kill operations across shards if individual shards operation dies

### Sharding: Latency improvements on Secondary Reads

- Enabled time stamps in the storage Engine
- Transactions have consistent view of data at a specific cluster time -> guarantees a consistent view of the data

## Chapter 3: Misc Server Improvements

### TLS 1.2 Support

- Payment Card Industry Data Security Standard (PCI DSS) recommands >1.1
- Your data should be securely encrypted in transit to prevent unauthorized access to it via the network.
- Without it, you may fail to meet corporate security policies which can prevent your applications from going into production.
- TLS certificates can be used as a means of securely confirming the identity of an entity attempting to access your data.

### SCRAM-SHA-256

- Remove MongoDB CR Authenication Mechanism (SHA-1)
- New user will use SHA-256 by default : the createUser command is the same
- To migrate existing user, reset the password. But admin doesn't know these password, so users have to do this.

### Change Stream Improvements

- Stream is a functionality included in MongoDB 3.6
- Database level changeStream : `db.watch()`
- Cluster level changeStream : `db.getMongo().watch()`
- startAtOperationTime Option : `db.A.watch({startAtOperationTime: t10 })`
- Refined Change Event syntax that support transactions

## Chapter 4: Aggregation Framework Improvements

### Type Conversions

```json
{
    "$convert": {
        "input": <any-expression>,
        "to": <type> | <expression evaluating to type>,
        "onError": <any-expression> /* Optional */,
        "onNull": <any-expression> /* Optional */
    }
}
```

Example of `$dateFromPart` :

```js
db.address.aggregate([
  {
    $addFields: {
      next_visit: {
          $convert:{
            input: {
              $dateFromParts: {
                year: "$last_visited.year",
                month: {$add:[15, "$last_visited.month"]},
              }},
            to: "date",
            onNull: "",
            onError: ""
        }
      }
    }
  }
])
```

Example of `$trim` :

```js
db.address.aggregate( [
 { $match: { building: {$type: "string"} }},
 {
   $addFields: {
     building: {
        $convert: {
          input: {$trim: {input: "$building", chars: "w"}},
          to: "int"
        }
      }
    }
 },
 {$sort: {building: 1}}  ])
```

## Chapter 5: MongoDB Compass

### Introduction to new Features

- Aggregation pipeline builder
- Import/Export data in JSON or CSV formats
- Export query in Language : JavaScript, Java, Python and C#
