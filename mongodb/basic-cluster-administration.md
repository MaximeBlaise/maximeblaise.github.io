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

### File Structure

```powershell
ls -l /data/db                  # list --dbpath directory
ls -l /data/db/diagnostic.data  # list diagnostics data directory
ls -l /data/db/journal          # list journal directory
ls -l /tmp/mongodb-27017.sock   # list socket file
```

Each collection and index has its own `.wt` file.
Warning: not readable for human. Not modify it yourself

### Basic Commands

Basic Helper Groups :

- db.<method>(): interact with databases
  - db.<collection>.<method>()
- rs.<method>(): control replica sets
- sg.<method>(): control sharded cluster

Basic Commands

```powershell
# User management commands:
db.createUser()
db.dropUser()

# Collection management commands:
db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()

# Database management commands:
db.dropDatabase()
db.createCollection()

# Database status command:
db.serverStatus()

# Creating index with Database Command:
db.runCommand(
  { "createIndexes": <collection> },
  { "indexes": [
    {
      "key": { "product": 1 }
    },
    { "name": "name_index" }
    ]
  }
)

# Creating index with Shell Helper:
db.<collection>.createIndex(
  { "product": 1 },
  { "name": "name_index" }
)

# Introspect a Shell Helper:
db.<collection>.createIndex
```

### Logging Basics

```powershell
# Get the logging components:
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.getLogComponents()
'

# Change the logging level:
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.setLogLevel(0, "index")
'

# Tail the log file:
tail -f /data/db/mongod.log

# Update a document:
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.products.update( { "sku" : 6902667 }, { $set : { "salePrice" : 39.99} } )
'

# Look for instructions in the log file with grep:
grep -R 'update' /data/db/mongod.log
```

Log Verbosity Levels :

N° | Description
--- | ---
-1 | Inherit from parent
0 | Default Verbosity, to include Informational messages
1 - 5 | Increases the verbosity level to include Debug messages

Log Message Severity Levels :

Level | Description
--- | ---
F | Fatal
E | Error
W | Warning
I | Informational (Verbosity Level 0)
D | Debug (Verbosity Level 1-5)

### Profiling the Database

Events captured by the profiler :

- CRUD
- Administrative operations
- Configuration operations

Level | Description
--- | ---
0 | The profiler is off and does not collect any data. (Default)
1 | The profiler collects data for operations that take longer than the value of slowms.
2 | The profiter collects data for all operations

```powershell
db.getProfilingLevel()
db.setProfilingLevel(1)
show collections        #  show system.profile collection

db.setProfilingLevel( 1, { slowms: 0 } )
db.new_collection.insert( { "a":1 } )
db.system.profile.find().pretty()  # show lots of information
```

Exemple of configuration file :

```yaml
storage:
  dbPath: "/var/mongodb/db/"
systemLog:
  path: "/var/mongodb/db/mongod.log"
  destination: "file"
  logAppend: true
net:
  bindIp : "localhost,192.168.103.100"
  port: 27000
processManagement:
  fork : true
operationProfiling:
  slowOpThresholdMs: 50
```

### Basic Security

Authentication | Authorization
--- | ---
Verify the **Identity** of a user | Verifies the **privileges** of a user
Answers the question : **Who are you ?** | Answers the question : **What do you have access to?**

Authentication Mechanisms :

- SCRAM: Salted Challenge Response Authentication Mechanism
- X.509
- LDAP: Enterprise only (AD)
- Kerberos : Entreprise only

There is also cluster authentication.
Always configure at least SCRAM. To learn more about Security, please visit course M310 at mongoDB University

Role Bases Access Protocol :

- Database administrator: Create user/index
- Developer: Write/Read data
- Data scientist: Read data

```powershell
# Print configuration file:
cat /etc/mongod.conf # show security.authorization: enabled

# Launch standalone mongod:
mongod -f /etc/mongod.conf

# Connect to mongod:
mongo --host 127.0.0.1:27017

# Create new user with the root role (also, named root):
use admin
db.createUser({
  user: "root",
  pwd: "root123",
  roles : [ "root" ]
})

# Connect to mongod and authenticate as root:
mongo --username root --password root123 --authenticationDatabase admin

# Run DB stats:
db.stats()

# Shutdown the server:
use admin
db.shutdownServer()
```

### Built-In Roles

How Role Bases Access Control (RBAC) works

- Built-in Roles
  - Pre-packaged MongoDB Roles
- Role Structure
  - Privileges:
  - Action + Resource

Resources can be :

- Database
- Collection
- Set of Collections
- Cluster
  - Replica Set
  - Shard Cluster

```json
// specific database and collection
{ db: "products", collection: "inventory" }

// all databases and all collections
{ db: "", collection: "" }

// any database and specific collection
{ db: "", collection: "accounts" }

// specitic database any collection
{ db: "products", collection: "" }

// or cluster resource
{ cluster: true}
```

Example of Privilege :

```json
// allow to shutdown over the cluster
{ resource: { cluster: true }, actions: [ "shutdown" ] }
```

Details of Built-In Roles :

- Database User: read/readWrite
- Database Administration: read/readWrite (dbAdmin, userAdmin, dbOwbner)
- Cluster administration: clusterAdmin, clusterManager, clusterMonitor, hostManager
- Backup/Restore
- Super User: root

Examples of commands :

```powershell
# Authenticate as root user:
mongo admin -u root -p root123

# Create security officer:
db.createUser(
  { user: "security_officer",
    pwd: "h3ll0th3r3",
    roles: [ { db: "admin", role: "userAdmin" } ]
  }
)

# Create database administrator:
db.createUser(
  { user: "dba",
    pwd: "c1lynd3rs",
    roles: [ { db: "admin", role: "dbAdmin" } ]
  }
)

# Grant role to user:
db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )

# Show role privileges:
db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} )
```