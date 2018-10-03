# Basic Cluster Administration

[Home](../README.md) > [MongoDB](./readme.md) > [Basic Cluster Administration](./basic-cluster-administration.md)

Units | Topics
--- | ---
[Chapter 0: Introduction](#chapter-0-introduction) | An overview of the course content.
[Chapter 1: The Mongod](#chapter-1-the-mongod) | Standalone node configuration and setup
[Chapter 2: Replication](#chapter-2-replication) | Basic replication concepts and replica set administration
[Chapter 3: Sharding](#chapter-3-sharding) | Sharded cluster creation and management

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

```powershell
db.<method>() # interact with databases
  db.<collection>.<method>()
rs.<method>() # control replica sets
sg.<method>() # control sharded cluster
```

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

### Server Tools Overview

```powershell
# List mongodb binaries:
find /usr/bin/ -name "mongo*"

# Create new dbpath and launch mongod:
mkdir -p ~/first_mongod
mongod --port 30000 --dbpath ~/first_mongod --logpath ~/first_mongod/mongodb.log --fork

# Use mongostat to get stats on a running mongod process:
mongostat --port 30000

# Use mongodump to get a BSON dump of a MongoDB collection:
mongodump --port 30000 --db applicationData --collection products
ls dump/applicationData/
cat dump/applicationData/products.metadata.json # contains for example indexes

# Use mongorestore to restore a MongoDB collection from a BSON dump:
mongorestore --drop --port 30000 dump/

# Use mongoexport to export a MongoDB collection to JSON or CSV (or stdout!):
mongoexport --port 30000 --db applicationData --collection products -o products.json

# Use mongoimport to create a MongoDB collection from a JSON or CSV file:
mongoimport --port 30000 products.json
```

## Chapter 2: Replication

### What is Replication

MongoDB uses asynchronous, statement-bases replication.

Replication = Maintain multiple copies of your data.
Failover = if primary node is down, one of the secondaries can play the primary role.

For example we insert

```powershell
db.students.insert(
  { "name": "Matt Javaly", "grade", "A" }
)
```

Memory Address | Data to be written
--- | ---
0x100203 | "Matt Javaly"
0x3a123f | "A"

Two types of replication :

Binary Replication | Statement-Based Replication
--- | ---
The secondary node will receieve a copy of binary log<br/>But, operating system must be consistent across the entire replica set | Use oplog<br/>Regardless of operating system

Idempotence: we can execute oplog several times, the data have to be at the same state.

### MongoDB Replica Set

Read more about the [Simple Raft Protocol](http://thesecretlivesofdata.com/raft/) and the [Raft Consensus Algorithm](https://raft.github.io/).

The secondaries will get data from primary, through an asynchronous replication mechanism.

Arbiter:

- holds no data
- can vote in an election
- cannot becone primary

There is a failover mechanism in place: if we lose primary node, we cannot write anymore. Which of the secondaries could become the new primary ?

Replica Set: Up to 50 Members, 7 voting Members

### Setting Up a Replica Set

Read more about [VI commands](http://www.lagmonster.org/docs/vi.html).

The configuration file for the first node (nodeN.conf):

```yaml
storage:
  dbPath: /var/mongodb/db/nodeN
net:
  bindIp: 192.168.103.100,localhost
  port: 2701N
security:
  authorization: enabled
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/nodeN/mongod.log
  logAppend: true
processManagement:
  fork: true
replication:
  replSetName: m103-example
```

```powershell
# Creating the keyfile and setting permissions on it:
sudo mkdir -p /var/mongodb/pki/
sudo chown vagrant:vagrant /var/mongodb/pki/
openssl rand -base64 741 > /var/mongodb/pki/m103-keyfile
chmod 400 /var/mongodb/pki/m103-keyfile

# Creating the dbpath for nodeN:
mkdir -p /var/mongodb/db/nodeN

# Starting a mongod with nodeN.conf:
mongod -f nodeN.conf

# Connecting to node1:
mongo --port 27011

# Initiating the replica set:
rs.initiate()

# Creating a user:
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})

# Exiting out of the Mongo shell and connecting to the entire replica set:
exit
mongo --host "m103-example/192.168.103.100:27011" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"

# Getting replica set status:
rs.status()

# Adding other members to replica set:
rs.add("m103.mongodb.university:27012")
rs.add("m103.mongodb.university:27013")

# Getting an overview of the replica set topology:
rs.isMaster()

# Stepping down the current primary:
rs.stepDown()

# Checking replica set overview after election:
rs.isMaster()
```

### Replication Configuration Document

- JSON Object that defines the configuration options of our replica set
- Can be configured manually from the shell
- There are set of mongo shell replication helper methods that make it easier to manage

```powershell
rs.add
rs.initiate
rs.remove
rs.reconfig
rs.config
```

```json
{
  _id: "string",
  version: "int",
  members: [
    {
      _id: "int",
      host: "string",
      arbiterOnly: "boolean",
      priority: "number",
      slaveDelay: "int" // in seconds
    },
    ...
  ]
}
```

### Replication Commands

```powershell
rs.status()
  # Reports health on replica set nodes
  # Uses data from heartbeats

rs.isMaster()
  # Describes a nodes's role in the replica set
  # Shorter output than rs.status()

db.serverStatus()['repl']
  # Section of the db.serverStatus() output
  # Similar to the output of rs.isMaster()

rs.printReplicationInfo()
  # Only returns oplog data relative to current node
  # Contains timestamps for first and last oplog events
```

### Local DB

```powershell
# Making a data directory and launching a mongod process for a standalone node:
mkdir allbymyselfdb
mongod --dbpath allbymyselfdb

# All MongoDB instances start with two default databases, admin and local:
mongo
show dbs

# Display collections from the local database (this displays more collections from a replica set than from a standalone node):
use local
show collections

# Querying the oplog after connected to a replica set:
use local
db.oplog.rs.find()
```

```powershell
# Storing oplog stats as a variable called stats:
var stats = db.oplog.rs.stats()

# Verifying that this collection is capped (it will grow to a pre-configured size before it starts to overwrite the oldest entries with newer ones):
stats.capped

# Getting current size of the oplog:
stats.size

# Getting size limit of the oplog:
stats.maxSize

# Getting current oplog data (including first and last event times, and configured oplog size):
rs.printReplicationInfo()
```

```powershell
# Create new namespace m103.messages:
use m103
db.createCollection('messages')

# Query the oplog, filtering out the heartbeats ("periodic noop") and only returning the latest entry:
use local
db.oplog.rs.find( { "o.msg": { $ne: "periodic noop" } } ).sort( { $natural: -1 } ).limit(1).pretty()

# Inserting 100 different documents:
use m103
for ( i=0; i< 100; i++) { db.messages.insert( { 'msg': 'not yet', _id: i } ) }
db.messages.count()

# Querying the oplog to find all operations related to m103.messages:
use local
db.oplog.rs.find({"ns": "m103.messages"}).sort({$natural: -1})

# Illustrating that one update statement may generate many entries in the oplog:
use m103
db.messages.updateMany( {}, { $set: { author: 'norberto' } } )
use local
db.oplog.rs.find( { "ns": "m103.messages" } ).sort( { $natural: -1 } )

# Remember, even though you can write data to the local db, you should not.
```

### Reconfiguring a Running Replica Set

Create node4.conf and arbiter.conf from chapter above "Setting Up a Replica Set".

```powershell
# Starting up mongod processes for our fourth node and arbiter:
mongod -f node4.conf
mongod -f arbiter.conf

# From the Mongo shell of the replica set, adding the new secondary and the new arbiter:
rs.add("m103.mongodb.university:27014")
rs.addArb("m103.mongodb.university:28000")

# Checking replica set makeup after adding two new nodes:
rs.isMaster()

# Removing the arbiter from our replica set:
rs.remove("m103.mongodb.university:28000")

# Assigning the current configuration to a shell variable we can edit, in order to reconfigure the replica set:
cfg = rs.conf()

# Editing our new variable cfg to change topology - specifically, by modifying cfg.members:
cfg.members[3].votes = 0
cfg.members[3].hidden = true
cfg.members[3].priority = 0

# Updating our replica set to use the new configuration cfg:
rs.reconfig(cfg)
```

### Reads and Writes on a Replica Set

```powershell
# Connecting to the replica set:
mongo --host "m103-example/m103.mongodb.university:27011" -u "m103-admin" -p
"m103-pass" --authenticationDatabase "admin"

# Checking replica set topology:
rs.isMaster()

# Inserting one document into a new collection:
use newDB
db.new_collection.insert( { "student": "Matt Javaly", "grade": "A+" } )

# Connecting directly to a secondary node (this node may not be a secondary in your replica set!):
mongo --host "m103.mongodb.university:27012" -u "m103-admin" -p "m103-pass"
--authenticationDatabase "admin"

# Attempting to execute a read command on a secondary node (this should fail):
show dbs

# Enabling read commands on a secondary node:
rs.slaveOk()

# Reading from a secondary node:
use newDB
db.new_collection.find()

# Attempting to write data directly to a secondary node (this should fail, because we cannot write data directly to a secondary):
db.new_collection.insert( { "student": "Norberto Leite", "grade": "B+" } )

# Shutting down the server (on both secondary nodes)
use admin
db.shutdownServer()

# Connecting directly to the last healthy node in our set:
mongo --host "m103.mongodb.university:27011" -u "m103-admin" -p "m103-pass"
--authenticationDatabase "admin"

# Verifying that the last node stepped down to become a secondary when a majority of nodes in the set were not available:
rs.isMaster()
```

### Failover and Elections

```powershell
# Storing replica set configuration as a variable cfg:
cfg = rs.conf()

# Setting the priority of a node to 0, so it cannot become primary (making the node "passive"):
cfg.members[2].priority = 0

# Updating our replica set to use the new configuration cfg:
rs.reconfig(cfg)

# Checking the new topology of our set:
rs.isMaster()

# Forcing an election in this replica set (although in this case, we rigged the election so only one node could become primary):
rs.stepDown()

# Checking the topology of our set after the election:
rs.isMaster()
```

### Write Concerns

Levels | Description
--- | ---
0 | Don't wait for acknowledgement
1 (default) | Wait for acknowledgement from the primary only
>= 2 | Wait for acknowledgement from the primary and one or more secondaries
`majority` | wait for acknowledgement from a majority of replica set members

Write Concerns Options :

- `wtimeout`: (int) the time to wait for the requested write concern before making the operation as failed
- `j`: (boolean) requires the node to commit the write operation to the journal before returning an acknowledgement

Write Concerns Commands :

- insert
- update
- delete
- findAndModify

### Read Concerns

Read concern provides a way of dealing with the issue of data durability during a failover event.

```powershell
db.products.find(
  { "name": "things", ... }
).readConcern( level: "majority" )
```

Levels | Feature | Description
--- | --- | ---
local (default) | Latest & Fast | most recent data in the cluster
available (sharded clusters) | Latest & Fast | default with read from secondaries nodes
majority | Safe & Fast | returns data acknowledged as written to a majority of replica set members
linearizable | Safe & Latest | returns data from write operations with write concern of majority

### Read Preferences

```powershell
db.products.find(
  { "name": "Mongo 101", ... }
).readPref("secondaryPreferred")
```

Read Preference Modes

- primary (default)
- primaryPreferred
- secondary
- secondaryPreferred
- nearest (least network latency)

Scenario | Tradeoff | Read Preference
--- | --- | ---
Read from the primary only | Secondaries are for availability only | primary (default)
If the primary is unavailable, read from a secondary | Possible to read stale data | primaryPreferred
Read from the secondary members only | Possible to read stale data | secondary
If all secondaries are unavailable, read from the primary | Possible to read stale data | secondaryPreferred
Application's read from the geographically closest member | Possible to read stale data | nearest

## Chapter 3: Sharding

### What is sharding

It's possible to store all mongo in the same server.

**Vertical Scaling**
If the performance is not enough, one possibility is just to make the machine better (more RAM, etc.)
This solution can become very expensive.

**Horizontal Scaling**
Add more machines and then distribute the dataset

The way the data is distributed in MongoDB is called `Sharding`.

_Mongos_: the process which route query into the right Shard. It uses metadata and Config Server.

### When to Shard

Check first if it is still economically viable to vertical scale.

Warning when increasing disk spaces. It takes more time to backup & restore.
Good practice: between 2TB and 5TB / server

If Aggregation pipelines become too slow, it's time to Shard.
Check M121 - MongoDB Aggregation Course

Zone Sharding allow us to easily distribute data that needs to be co-located.

### Sharding Architecture

We can add any number of Shards.
Clients don't communicate directly with the shard => mongos (route process).

Example of Sharded cluster (with Football players) metadata :

Shard | Data
--- | ---
1 | A-J
2 | K-Q
3 | R-Z

Primary Shard.
The mongos is responsible of Aggregation Commands (SHARD_MERGE).

### Setting Up a Sharded Cluster

Configuration file for first config server csrs_N.conf:

```yaml
sharding:
  clusterRole: configsvr
replication:
  replSetName: m103-csrs
security:
  keyFile: /var/mongodb/pki/m103-keyfile
net:
  bindIp: localhost,192.168.103.100
  port: 2600N
systemLog:
  destination: file
  path: /var/mongodb/db/csrsN.log
  logAppend: true
processManagement:
  fork: true
storage:
  dbPath: /var/mongodb/db/csrsN
```

mongos.conf

```yaml
sharding:
  configDB: m103-csrs/192.168.103.100:26001,192.168.103.100:26002,192.168.103.100:26003
security:
  keyFile: /var/mongodb/pki/m103-keyfile
net:
  bindIp: localhost,192.168.103.100
  port: 26000
systemLog:
  destination: file
  path: /var/mongodb/db/mongos.log
  logAppend: true
processManagement:
  fork: true
```

Commands about config servers :

```powershell
mongod -f csrs_1.conf
mongo --port 26001
rs.initiate()

# Create super user on CSRS
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})

# Authenticating as the super user:
db.auth("m103-admin", "m103-pass")

# Add the second and third node to the CSRS:
rs.add("192.168.103.100:26002")
rs.add("192.168.103.100:26003")

mongos -f mongos.conf
mongo --port 26000 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
# Check sharding status:
sh.status()
```

Updated configuration for node1.conf:

```yaml
sharding:
  clusterRole: shardsvr
storage:
  dbPath: /var/mongodb/db/nodeN
  wiredTiger:
    engineConfig:
      cacheSizeGB: .1
net:
  bindIp: 192.168.103.100,localhost
  port: 2701N
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/nodeN/mongod.log
  logAppend: true
processManagement:
  fork: true
replication:
  replSetName: m103-repl
```

```powershell
# Connecting directly to secondary node (note that if an election has taken place in your replica set, the specified node may have become primary):
mongo --port 27012 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
mongod -f node2.conf

# Stepping down current primary:
mongo --port 27011 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.stepDown()

# Adding new shard to cluster from mongos:
sh.addShard("m103-repl/192.168.103.100:27012")
```

### Config DB

Never write data into `config` database.

```powershell
use config
db.databases.find().pretty()    # Query config.databases
db.collections.find().pretty()  # Query config.collections
db.shards.find().pretty()       # Query config.shards
db.chunks.find().pretty()       # Query config.chunks
db.mongos.find().pretty()       # Query config.mongos
```

### Shard Keys

- Shard Key Fields mus tbe indexed
  - Indexes must exist first before you can select the indexed fields for your shard key
- Shard Keys are **immutable**
  - You cannot change the shard key fields post-sharding
  - You cannot change the values of the shard key fields post-sharding
- Shard keys are **permanent**
  - You cannot unshard a sharded collection

```powershell
use m103
show collections
sh.enableSharding("m103")
db.products.createIndex( { "sku" : 1 } ) # Create an index on sku
sh.shardCollection("m103.products", {"sku" : 1 } ) # Shard the products collection on sku
sh.status() # Checking the status of the sharded cluster
```

### Picking a Good Shard Key

The goal is a shard key whose values provides good write distribution

- Cardinality
  - High cardinality = many possible unique shard key values
- Frequency
  - Low Frequency = Low repetition of a given unique shard key value
- Monotonic Change
  - Timestamp for exemple always increase, always go to the last shard cluster.

Read Isolation : direct read into the right Shard.

### Hashed Shard Keys

It's possible to have Hashed Shard Keys :

- Queries on ranges of shard key values are more likely to be scatter-gather
- Cannot support geographically isolated read operations using zoned sharding
- Hashed Index must be on a single non-array field
- Hashed Indexes don't support fast sorting

Sharding using a Hashed Shard Key :

```powershell
sh.enableSharding("database")
db.collection.createIndex("field": "hashed")
sh.shardCollection( "database.collection", { "shard key field", "hashed" } )
```

### Chunks

Chanks are logical groups of document.

```powershell
# Find one document from the chunks collection:
use config
db.chunks.findOne()

# Change the chunk size:
use config
db.settings.save({_id: "chunksize", value: 2})

# Check the status of the sharded cluster:
sh.status()

# Import a new dataset (note that this dataset does not exist on the course VM):
mongoimport -d m103 -c products products.part2.json
```

Jombo Chunks

- Larger than the defined chunk size
- Cannot move jumbo chunks
  - One marked as jombo the balancer skips these chunks and aboids trying to move them
- In some cases these will not be able to be split

### Balancing

Balancers can split Chunks if needed.

In the mongos :

```powershell
sh.startBalancer(timeout, interval) # Start the balancer
sh.stopBalancer(timeout, interval) # Stop the balancer
sh.setBalancerState(boolean) # Enable/disable the balancer
```

### Queries in a Sharded Cluster

- **mongos** handles all queries in the cluster
- **mongos** builds a list of shards to target a query
- **mongos** merges the results from each shard
- **mongos** supports standard query modifies like `sort`, `limit` and `skip`.

`sort()`: the mongos pushes the sort to each shard and merge-sorts the results
`limit()`: the mongos passes the limit to each targeted shard, then re-applies the limit to be merged set of results
`skip()`: the mongos performs the skip against the merged set of results

### Routed Queries vs Scatter Gather

Shard Key : `{ "sku": 1, "type": 1, "name": 1 }`

```powershell
# Targetable queries :
db.products.find( { "sku": "..." } )
db.products.find( { "sku": "...", "type": "..." } )
db.products.find( { "sku": "...", "type": "...", "name": "..." } )

# Scatter-gather queries :
db.products.find( { "type": "..." } )
db.products.find( { "name": "..." } )

# Routed query with explain() output:
db.products.find({"sku" : 1000000749 }).explain()

# Scatter gather query with explain() output:
db.products.find( { "name" : "Gods And Heroes: Rome Rising - Windows [Digital Download]" } ).explain()
```
