# MongoDB for Python Developers

[Home](../README.md) > [MongoDB](./readme.md) > [MongoDB for Python Developers](./python.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | Course logistics, requirements for environment setup, and application architecture.
[Chapter 1: Driver Setup](#chapter-1-driver-setup) | Database client configuration, basic reads.
[Chapter 2: User-Facing Backend](#chapter-2-user-facing-backend) | Basic aggregation, updates, deletes, and joins.

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
