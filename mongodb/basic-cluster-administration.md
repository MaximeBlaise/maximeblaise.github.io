# Basic Cluster Administration

[Home](../README.md) > [MongoDB](./readme.md) > [Basic Cluster Administration](./basic-cluster-administration.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | An overview of the course content.
[Chapter 1: The Mongod](#chapter-1-the-mongod) | Standalone node configuration and setup

## Chapter 0: Introduction

### Introduction to Replication and Sharding

Topics covered :

- Architecture of a sharded cluster
- Query handling in a sharded cluster
- Data distribution method

### Setting Up the Vagrant Environment

Why are we using a virtual environment ?

- Sandbox environment for the course
- Avoid dependency and system troubleshooting
  - Focus on learning
- Provide more consistent support

```powershell
vagrant up
vagrant provision
vagrant ssh
```

Vagrant files can be found in [MongoDB University courses](https://university.mongodb.com/).

## Chapter 1: The Mongod

### The Mongod

The Mongod is the main daemon process for MongoDB. To run the daemon, just enter the following command :

```powershell
> mongod
```

You will see a bunch of output :

```powershell
[initandlisten] MongoDB starting : pid=21824 port=27017 dbpath=C:\data\db\ 64-bit host=COMPUTERNAME
[...]
[initandlisten] waiting for connections on port 27017
```

Use of mongo shell :

```powershell
> mongo # Connect to the Mongo Shell
> mongo admin --eval 'db.shutdownServer()' # Shutdown Mongod
> exit # Exit the Mongo Shell
```

Some useful command about Mongod :

```powershell
> mongod --help # list of flags

# Start new daemon
> mkdir first_mongod
> mongod --port 30000 --dbpath first_mongod --logpath first_mongod/mongod.log --fork

> mongo --port 30000
> mongo admin --port 30000 --eval 'db.shutdownServer()'
```

You can read more about the MongoDB server in the [daemon documentation](https://docs.mongodb.com/manual/reference/program/mongod).

### MongoDB Architecture

- MongoDB Query Language (MQL)
- MongoDB Document Data Model
  - namespaces
  - indexes
  - data structures
  - replication mechanism : WriteConcerns | ReadConcerns
- Storage Layer : **WiredTiger**, Encrypted, In-Memory, MMapV1
  - system calls
  - disk flush
  - file structures
  - compression
- Security / Administration

High Availability & Failover (Primary/Secondary, Raft protocol)

Sharded Clusters provide scalability
