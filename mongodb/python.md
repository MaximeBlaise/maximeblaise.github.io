# MongoDB for Python Developers

[Home](../README.md) > [MongoDB](./readme.md) > [MongoDB for Python Developers](./python.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | Course logistics, requirements for environment setup, and application architecture.
[Chapter 1: Driver Setup](#chapter-1-driver-setup) | Database client configuration, basic reads.

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
