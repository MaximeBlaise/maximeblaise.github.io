# New Features and Tools in MongoDB 4.0

[Home](../README.md) > [MongoDB](./readme.md) > [New Features and Tools in MongoDB 4.0](./new-in-4.0.md)

Units | Topics
--- | ---
[Chapter 1: Replica Set Transactions](#chapter-1--replica-set-transactions) | Replica Set Transactions: a general overview on transaction support, detailed configuration options, and use cases.
[Chapter 2: Sharding Improvements](#chapter-2--sharding-improvements) | Sharding Improvements: updates to sharding metadata configuration commands, routing and balancing improvements, and more detailed logging information on mongos
[Chapter 3: Misc Server Improvements](#chapter-3-misc-server-improvements) | Security and Profiling Miscellaneous: SCRAM-SHA-256, TLS 1.2, ChangeStreams improvements, and better profiling facilities.
[Chapter 4: Aggregation Framework Improvements](#chapter-4-aggregation-framework-improvements) | Aggregation Framework Improvements: new type conversion expressions with syntactic sugar operators.
[Chapter 5: MongoDB Compass](#chapter-5-mongodb-compass) | MongoDB Compass new features and functionality: aggregation pipeline builder, export to language, JSON Schema validation builder.
[Chapter 6: ODBC Driver for BI Connector](#chapter-6-odbc-driver-for-bi-connector) | BI Connector: New ODBC driver and improved installation & configuration.
[Chapter 7: Atlas](#chapter-7-atlas) | MongoDB Atlas features from the last 6 months: cross region clusters, multi-region writes, data explorer, and live migration.
[Chapter 8: MongoDB Upgrade and Downgrade](#chapter-8-mongodb-upgrade-and-downgrade) | MongoDB 4.0 Upgrade Tutorial: the steps necessary to correctly upgrade to MongoDB 4.0 and how to downgrade
[Chapter 9: Ops Manager 4.0](#chapter-9-ops-manager-40) | MongoDB Ops Manager 4.0 Kubernetes integration

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

## Chapter 6: ODBC Driver for BI Connector

### Windows - Installing ODBC Driver

You can find a link to download ODBC Driver [here](https://github.com/mongodb/mongo-odbc-driver/releases/tag/v1.0.0)
Just download needed file (x32 or x64), and install it by double-clicking in it.

### Windows - Configuring DSN

- Launch `ODBC datasource administrator`
- Click `Add` and select `MongoDB ODBC 1.0 Unicode Driver`
- Fill in connection parameters

## Chapter 7: Atlas

### Recent Features - Cloud Providers, Multi Regions

- Support for the top 3 Cloud providers
- Spreading a cluster across many regions
- Upcoming feature Global Sharding for geographical access of documents

- Activity Feed at the Organization level
- Database level auditing (premium)
- Using your private LDAP directory
- Using your private KMS for encryption keys

- Compliance with GDPR standards

### Hosted BI Connector, Upgrading from Free Tier

- Using the hosted BI connector in Atlas (premium)
- Upgrading a cluster from Free Tier to larger paid instances
- Using MongoDB 4.0 in Atlas

### Cloud Provider Snapshot Backups

### Real Time Panel, CRUD in Atlas UI

- Real time performance panel : see slow operations and kill them directly
- CRUD operations within the Atlas UI

## SRV records, Data Migrator

- Support for SRV records in connection strings
- Command line tools
- Data migrator to Atlas
- Test failover
- Cross project restores

## Auto Scaling Storage, Performance Advisor

- Auto scaling storage capacity
- Performance advisor
- Region migration

## Chapter 8: MongoDB Upgrade and Downgrade

### Downgrade

Procedure for a Replica Set :

- Remove any dependencies on 4.0 features that are not backwards compatible
- Set *featureCompatibilityVersion* to `3.6`
- Connect to one secondary node at a time, and :
  - Shut down the secondary
  - Start up secondary node using the 3.6 binary
- When all secondaries are online, connect to the primary and:
  - Step down the primary and wait for elections to complete
  - Shut down the (now old) primary
  - Start up the node using the 3.6 binary
- Confirm all members are communicating successfully

List mongod nodes:

`ps -ef | grep mongod`

Then, connect with mongo shell:

```js
rs.status() // Check replica set status
db.adminCommand( { setFeatureCompatibilityVersion: "3.6" } ) // Set feature compatibility mode
```

Downgrade secondary nodes: (just replace .conf files)

```bash
sudo ln -s /opt/mongodb/mongodb-linux-x86_64-enterprise-ubuntu1404-3.6.5/bin/mongod /usr/bin/mongod3.6
mongo admin --host 127.0.0.1:27027 --eval 'db.shutdowServer()'
ps -ef | grep mongod
mongod3.6 -f /shared/rs2_unix.conf
# check status
mongo admin --host 127.0.0.1:27027 --eval 'rs.slaveOk();rs.status()'
ps -ef | grep mongod
# the remaining secondary
mongo admin --host 127.0.0.1:27037 --eval 'db.shutdowServer()'
ps -ef | grep mongod
mongod3.6 -f /shared/rs3_unix.conf
ps -ef | grep mongod
```

Connect to primary node:

```bash
mongo admin --host 127.0.0.1:27017
```

Step down and shutdown server:

```js
use admin
rs.stepDown()
rs.status()
db.shutdownServer()
```

Downgrade the old primary

```bash
mongod3.6 -f /shared/rs1_unix.conf
mongo --host M040/m040:27017,m040:27027,m040:27037 --eval 'rs.status()'
```

### Upgrade to MongoDB 4.0

Main considerations when Upgrading :

- Compatibility changes
  - Removed functionality in 4.0:
    - Support for MONGODB-CR
    - Support for Protocol Version 0 (PV0)
    - Support for TLS 1.0
    - Not using Journaling with WiredTiger
    - Master-Slave Replication
  - Significant change in behavior in 4.0 versus previous 3.6 release

- Backward Incompatible Changes
  - New or modified features of 4.0 that would not run with 3.6 or earlier
    - Multi-document transactions
    - Support for SCRAM-SHA-256
    - Type conversion operators and enhancements
    - `$dateToString` option changes

To upgrade to 4.0, first upgrade to `MongoDB 3.6` first.

## Chapter 9: Ops Manager 4.0

### Ops Manager Kubernetes Integration - Introduction

Kubernetes (k8S) = open source platform for managing containerized workloads and services

### Ops Manager Kubernetes Integration - Setup

Dependencies

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [docker](https://docs.docker.com/install/)
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [helm](https://docs.helm.sh/using_helm/#installing-helm)

Check required software dependencies

```powershell
virtualbox --help
docker version
kubectl version
minikube version
helm version
```

Startup minikube

```powershell
minikube start --memory="10000"
minikube status
eval $(minikube docker-env)
docker images
```

### Kubernates Integration

```bash
kubectl create namespace mongodb
kubectl config set-context $(kubectl config current-context) --namespace=mongodb
kubectl get all

# clone the repository
git clone https://github.com/mongodb/mongodb-enterprise-kubernetes

kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm init --service-account tiller

# look for the instructions on how to create a secret
cat mongodb-enterprise-kubernetes/README.md

kubectl create -f YOUR_SECRET.yml --namespace=mongodb

helm install

kubectl get all

kubectl logs -f

# download the simple test yaml configuration
curl -OL https://raw.githubusercontent.com/jasonmimick/mongodb-openshift-dev-preview/master/simple-test-opsmanager-k8s/simple-test-opsmgr.yaml
kubectl create -f simple-test-opsmgr.yaml

kubectl get all

 pod=$(kubectl get pods --selector=app=mongodb-opsmgr --output=jsonpath='{.items[0].metadata.name}')

 echo $pod

 kubectl exec -it $pod -c mongodb-opsmgr -- tail -f /mongodb-opsmgr-server/runtime/startup-mms.log
```

```bash
kubectl get all

# establish a bash shell within the container
kubectl exec -it $pod -c mongodb-opsmgr -- /bin/bash

ls /mongodb-opsmgr-server/runtime/
cd /mongodb-opsmgr-server/runtime/
cat operator-project-credenatials.yaml

kubectl exec -it $pod -c mongodb-opsmgr -- cat /mongodb-opsmgr-server/runtime/operator-project-credenatials.yaml | kubectl create -f -

kubectl get configmap

# create replica set deployment using yaml file
kubectl create -f m040-replica-set.yaml

kubectl get all

minikube ip

kubectl get svc

mongo mongodb://$(minikube ip):31529


kubectl exec -it -c mongodb-opsmgr -- /bin/bash

cd /mognodb-opsmgr/mongodb/bin
```