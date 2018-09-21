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

### Data Structures

Databases : top level containers

```powershell
db.createCollection()
db.createUser()
db.dropUser()
db.runCommand()

show dbs # show dabatases in mongod
use newDB # use an existing database
```

Collections : like tables.
If you run a query against a database/collection which not exist, MongoDB will create it for us.

```powershell
db.new_collection.insert( {a: 1} )
show colletions # list collections in specific database
```

Indexes : special data structure. See [M201: MongoDB Performance](https://university.mongodb.com/courses/M201/about) in MongoDB University.

### MongoDB Documents

Encode with JSON Format :

- Human readable and writable
- Machine parsable and generatable
- Language independant

```json
{
    "Firtname": "John",
    "Lastname": "Doe",
    "Age": 25
}
```

Firstname (string) | Lastname (string) | Age (int)
--- | --- | ---
John | Doe | 25

JSON supports documents in document, arrays, etc. :

```json
{
    "Firtname": "John",
    "Lastname": "Doe",
    "Age": 25,
    "Work": {
        "Status": "Employed",
        "Jobs": [
            "TotalJobs": 1,
            {
                "Company": "MongoFarms",
                "YearsWorked": 2
            }
        ]
    }
}
```

MongoDB JSON Limitations :

- Documents are restricted to 16MB in size

MongoDB use **BSON** (Binary JSON), which supports more Data types.

### Configuration File

Launch mongod with many configuration options:

```powershell
mongod --dbpath /data/db --logpath /data/log/mongod.log --fork --replSetName "M103" --keyFile /data/keyfile --bind_ip "127.0.0.1, 192.168.0.100" --sslMode requireSSL --sslCAFile "/etc/ssl/SSLCA.pem" --sslPEMKeyFile "/etc/ssl/ssl.pem"
```

Example configuration file, with the same configuration options as above:

```yaml
storage:
  dbPath: "/data/db"
systemLog:
  path: "/data/log.mongod.log"
  destination: "file"
replication:
  replSetName: M103
net:
  bindIp : "127.0.0.1, 192.168.0.10"
ssl:
  mode: "requireSSL"
  PEMKeyFile: "/etc/ssl/ssl.pem"
  CAFile: "/etc/ssl/SSLCA.pem"
security:
  keyFile: "/data/keyfile"
processManagement:
  fork : true
```

```powershell
mongod --config "/etc/mongod.conf"
mongod -f "/etc/mongod.conf"
```
