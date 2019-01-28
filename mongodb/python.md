# MongoDB for Python Developers

[Home](../README.md) > [MongoDB](./readme.md) > [MongoDB for Python Developers](./python.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | Course logistics, requirements for environment setup, and application architecture.
[Chapter 1: Driver Setup](#chapter-1-driver-setup) | Database client configuration, basic reads.
[Chapter 2: User-Facing Backend](#chapter-2-user-facing-backend) | Basic aggregation, updates, deletes, and joins.
[Chapter 3: Admin Backend](#chapter-3-admin-backend) | Read concerns and bulk operations.
[Chapter 4: Resiliency](#chapter-4-resiliency) | Connection pooling, error and timeout handling, and principle of least privilege.

## Chapter 0: Introduction

### Welcome to M220P

Technologies used :

- Python 3
- mongoDB
- Flask
- React : pre-build and bundled for us

### MFlix Application Architecture

`READMI.rst`: detailed setup instructions
`dotini`: contains information to connect to Atlas Cluster
`mflix/api`: contains some flask root handlers for the application
`mflix/build`: contains the front end application

`db.py`: our effort will be focused on ths file, contains all methods that interact with the database
`factory.py`: contains fonctionnality that assembles the FOS application for running. <- don't modify

## Chapter 1: Driver Setup

### Introduction to Chapter 1

- Basic read operations
- Adding field projections
- Handling fdiffernet query predicates
- Basic query semantics
- Basic shaping operations

### README

First, install anaconda from their [Windows download site](https://www.anaconda.com/download/#windows).

```powershell
# enter mflix-python folder
cd mflix-python

# create a new environment for MFlix
conda create --name mflix

# activate the environment
activate mflix

pip install -r requirements.txt

mongorestore --drop --gzip --uri mongodb+srv://m220student:m220password@<YOUR_CLUSTER_URI> data

mv dotini_unix .ini  # on Unix
ren dotini_win .ini # on Windows

python run.py
# This will start the application. You can then access the MFlix application at http://localhost:5000/.
```

### MongoClient

```python
from pymongo import MongoClient
uri = ""

client = MongoClient(uri)
client.stats
client.list_database_names()

mflix = client.mflix
mflix.list_collection_names()

mflix = client['mflix']
mflix.list_collection_names()

movies = mflix.movies
movies.count_documents({})

client = MongoClient(uri, connectTimeoutMS=200, retryWrites=True)
```

### Basic Reads

```python
import pymongo
uri = "mongodb+srv://m220-user:m220-pass@m220-lessons-mcxlm.mongodb.net/test"
client = pymongo.MongoClient(uri)
m220 = client.m220
movies = m220.movies

movies.find_one()
movies.find_one( { "cast": "Salma Hayek" } )
movies.find( { "cast": "Salma Hayek" } )
movies.find( { "cast": "Salma Hayek" } ).count()

cursor = movies.find( { "cast": "Salma Hayek" } )
from bson.json_util import dumps
print(dumps(cursor, indent=2)) # json library to display JSON

cursor = movies.find( { "cast": "Salma Hayek" }, { "title": 1 } )
print(dumps(cursor, indent=2))

cursor = movies.find( { "cast": "Salma Hayek" }, { "title": 1, "_id": 0 } )
print(dumps(cursor, indent=2))
```

## Chapter 2: User-Facing Backend

### Cursor Methods and Aggregation Equivalents

#### Limiting

You can limit the number of documents returned :

```python
limited_cursor = movies.find(
    { "directors", "Sam Raimi" },
    { "_id": 0, "title": 1, "cast": 1 }
).limit(2)

print(dumps(limited_cursor, indent=2))
```

The same things by using aggregation framework :

```python
pipeline = [
    { "$match": { "directors", "Sam Raimi" } },
    { "$project": { "_id": 0, "title": 1, "cast": 1 } },
    { "$limit": 2 }
]

limited_aggregation = movies.aggregate( pipeline )

print(dumps(limited_aggregation, indent=2))
```

#### Sorting

```python
from pymongo import DESCENDING, ASCENDING

sorted_cursor = movies.find(
    { "directors", "Sam Raimi" },
    { "_id": 0, "year": 1, "title": 1, "cast": 1 }
).sort("year", ASCENDING)

print(dumps(sorted_cursor, indent=2))
```

The same things by using aggregation framework :

```python
pipeline = [
    { "$match": { "directors", "Sam Raimi" } },
    { "$project": { "_id": 0, "year": 1, "title": 1, "cast": 1 } },
    { "$sort": { "year", ASCENDING } }
]

sorted_cursor = movies.aggregate( pipeline )

print(dumps(sorted_cursor, indent=2))
```

You have the possibility to sort with several keys :

```python
from pymongo import DESCENDING, ASCENDING

sorted_cursor = movies.find(
    { "directors", "Sam Raimi" },
    { "_id": 0, "year": 1, "title": 1, "cast": 1 }
).sort([("year", ASCENDING), ("title", ASCENDING)])

print(dumps(sorted_cursor, indent=2))
```

#### Skipping

```python
from pymongo import DESCENDING, ASCENDING

skipped_cursor = movies.find(
    { "directors", "Sam Raimi" },
    { "_id": 0, "year": 1, "title": 1, "cast": 1 }
).sort("year", ASCENDING).skip(10)

print(dumps(skipped_cursor, indent=2))
```

The same things by using aggregation framework :

```python
pipeline = [
    { "$match": { "directors", "Sam Raimi" } },
    { "$project": { "_id": 0, "year": 1, "title": 1, "cast": 1 } },
    { "$sort": { "year", ASCENDING } },
    { "$skip": 10 }
]

sorted_skipped_cursor = movies.aggregate( pipeline )

print(dumps(sorted_skipped_cursor, indent=2))
```

### Basic Aggragation

To learn more about the Aggregation Framework, check out [M121: The MongoDB Aggregation Framework](https://university.mongodb.com/courses/M121/about).

The Aggregation Pipeline in this lesson was produced using [MongoDB Compass](https://www.mongodb.com/products/compass).

Using Compass' [Aggregation Pipeline Builder](https://docs.mongodb.com/compass/master/aggregation-pipeline-builder/) feature, we can easily create, delete and rearrange the stages in a pipeline, and evaluate the output documents in real time. Then we can produce a version of that pipeline in Python, Java, C# or Node.js, using Compass' [Export-to-Language](https://docs.mongodb.com/compass/master/export-pipeline-to-language) feature.

- Aggregation is a pipeline
  - Pipelines are composed of stages, broad units of work.
  - Within stages, expressions are used to specify individual units of work
- Expressions are functions

Example with `add` function in different languages :

```python
def add(a, b):
    return a + b
```

```java
static <T extends Number> double add(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

```js
function add(a, b) {
    return a + b
}
```

```json
{ "$add": ["$a", "$b"] }
```

### Basic Writes

```python
import pymongo
uri = <your_uri>
client = pymongo.MongoClient(uri)
db.client.electronicsDB
db.collections_names()
vg = db.videos_games

insert_result = vg.insert_one({"title": "Fortnite", "year": 2018})
insert_result.acknowledged
insert_result.inserted_id

vg.find_one({ "_id": insert_result.inserted_id })
fortnite_doc = {"title": "Fortnite", "year": 2018}

upsert_result = vg.update_one( { "title": "Fortnite" }, { "$set": fortnite_doc}, upsert=True )
```

### Write Concerns

`writeConcern: { w:1 }`

- Only requests an acknowledgement that one node applied the write
- This is the dafault `writeConcern` in MongoDB

`writeConcern: { w:'majority' }`

- Requests acknowledgement that a **majority of nodes** in the replica set applied the write
- Takes longer than w: 1
- Is more durable than w: 1
  - Useful for ensuring vital writes are majority-committed

`writeConcern: { w:0 }`

- Does not request an acknowledgement that any ondes applied the write
  - "Fire-and-forget"
- Fastest writeConcern level
- Least durable writeConcern

### Basic Updates

`update_one`
`update_many`

```python
import pymongo
from bson.json_util import dumps
from faker import Faker
import random
fake = Faker()
fake.seed(42)
random.seed(42)
uri = <your_uri>
client = pymongo.MongoClient(uri)
mflix = client.mflix

fake_users = mflix.fake_users
fake_users.drop()
```

```python
def make_user(iter_count):
    account_type = "premium" if iter_count % 2 == 0 else "standard"
    return {
        "name": fake.name(),
        "address": fake.address(),
        "email": fake.email(),
        "age": random.randrange(18, 65),
        "favorite_colors": [fake.color_name(), fake.color_name(), fake.color_name()],
        "account_type": account_type
    }
```

```python
to_insert = [make_user(i) for i in range(10)]
fake_users.insert_many(to_insert)
print(dumps(fake_users.find_one(), indent=2))

allison = {"name": "Allison Hill"}
fake_users.update_one(allison, { "$inc": { "age": 1 }})
print(dumps(fake_users.find_one(allison), indent=2))

fake_users.update_one(allison, { "$push": { "favorite_colors": "Black" }})
print(dumps(fake_users.find_one(allison), indent=2))

u_r = fake_users.update_many({"account_type": "standard"}, {"$set": {"account_type": "premium", "free_trial": True} })
print(fake_users.count({"account_type": "standard"}))
print(dumps(fake_users.find({"free_trial": True}, {"_id": 0, "name": 1, "account_type": 1}), indent=2))
print(dir(u_r))
print(u_r.acknowledged, u_r..matched_count, u_r.modified_count, u_r.upserted_id) # True 5 5 None
```

### Basic Joins

- Join two collections of data
  - Movies and comments
- Use new expressive `$lookup`
- Build aggregation in Compass, and then export to language

1. Match stage

```json
{
    year: { 'gte': 1980, '$lt': 1990 }
}
```

2. Lookup stage

```json
{
    from: 'comments',
    let: { 'id': '$_id'},
    pipeline: [
        { '$match':
            { '$expr': { '$eq': [ '$movie_id', '$$id' ]}}
        },
        {
            '$count': 'count'
        }
    ],
    as: 'movie_comments'
}
```

### Basic Deletes

`delete_one` is like `find_one`
`delete_many` deletes all documents that match the supplied predicate.

## Chapter 3: Admin Backend

### Read Concerns

- Represent different levels of "read isolation"
- Can be used to specify a consistent view of the database

`local`: default MongoDB Read Concern. Read data when if it's not replicated yet.
`majority`: Verify that the data read is replicated to a majority of nodes in the set.

### Bulk Writes

Ordered Bulk Write (default)

- The default setting for bulk writes in MongoDB
- Executes writes sequentially
  - Will end execution after first write failure

Unordered Bulk Write

- Has to be specified with the flag: `{ ordered: false }`
- Executes writes in parallel

## Chapter 4: Resiliency

### Connection Pooling

Reusing database connection.
Connection pooling helps **reduce the overhead** of creating database connections.
Default size of 100.

[Documentation here](https://api.mongodb.com/python/current/api/pymongo/mongo_client.html)

### Robust Client Application

Always use connection pooling
Always specify a `wtimeout` with majority writes.

```json
{ w: "majority", wtimeout: 5000 }
```

Always configure for and handle `serverSelectionTimeout` errors.

### Writes with Error Handling

```python
# Initialisation
from pymongo import MongoClient, errors
uri = "<your_uri>"
mc = MongoClient(uri)
lessons = mc.lessons
shipments = lessons.shipments

shipments.find_one()
shipments.create_index("truck_id", unique=True)

try:
    res = shipments.insert_one(doc)
    print(res.inserted_id)
except errors.DuplicateKeyError:
    truck_id = doc["truck_id"]
    print(f"Truck #{truck_id} is currently performing a shipment. Please select another truck.")
```

### Principle of Least Privilege

Every program and every priviledged user of the system shoud operate using the least amount of privilege necessary to complete the job.

 - Jerome Saltzer, Communications of the ACM

### Change Streams

- Report changes at the collection level
- Accept pipelines to transform change events

`watch` method.
