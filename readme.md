## Introduction

Sample sharded cluster with SSL enabled

## Architecture

![Alt text](images/architecture.png "MongoDB cluster architecture")

## Pre-requisites

* MongoDB
* OpenSSL
* NodeJS
* Bash

## Step-1: Prepare SSL certificate

For production use, your MongoDB deployment should use valid certificates generated and signed by a single certificate authority.
You or your organization can generate and maintain an independent certificate authority, or use certificates generated by a
third-party SSL vendor. Obtaining and managing certificates is beyond the scope of this documentation.

For demonstration purpose you can generate [self sigined certificate.](./self-signed-certificate.md)

Generate a client certificate for web app & an admin-client for cluster administration, copy all certificates to `/opt/mongodb/`.
Set only read permission for these certificates.

```
chomod 400 *
```

## Step-2: Customize configs for your need

## Step-3: Initiate the cluster

Review before run the init script, also configure data directory path & directory permission.

```
bash init-shard.sh
```

This will setup all replica sets & Sharded cluster. Also will create two user webapp & appadmin.

View all mongod & mongos processes

```
px aux | grep mongo
```

## Step-4: Connect to a mongos query router

```
mongo --port 27018 --ssl --host database.fluddi.com --sslPEMKeyFile /opt/mongodb/admin-client.pem --sslCAFile /opt/mongodb/CA.pem
```

Authticate user
```
db.getSiblingDB("$external").auth(
  {
    mechanism: "MONGODB-X509",
    user: "emailAddress=support@fluddi.com,CN=*.fluddi.com,OU=appadmin,O=Fluddi,L=Dhaka,ST=Dhaka,C=BD"
  }
)
```

## Step-5: Make chunk size smaller

Make chunk size smaller for demonstration purpose otherwise, you will need to generate a huge volume of data.
This is only for demonstration purpose, don't do this in production.

```
use config
db.settings.save( { _id:"chunksize", value: <sizeInMB> } )
```

## Step-6: Generate some dummy data to the cluster

* Configure .env file, follow .env.example
* Install packages, use `npm i` or `yarn`
* Run `node index.js`, this will generate 50000 visitor records

## Others

### Check cluster status

```
sh.status()
```

### Gracefully shutdown cluster

Execute following commands from mongos
```
sh.stopBalancer()

# Use sh.getBalancerState() to verify that the balancer has stopped.
sh.getBalancerState()

# Now shutdown server
db.getSiblingDB("admin").shutdownServer()
```

### Start the sharded cluster
```
bash start.sh
```

### Change chunk size

```
use config
db.settings.save( { _id:"chunksize", value: <sizeInMB> } )
```

### Disable Balancing

```
sh.disableBalancing( "test.visitors")
```

### Create SSL a user

Example:
```
db.getSiblingDB("$external").runCommand(
  {
    createUser: "emailAddress=support@fluddi.com,CN=*.fluddi.com,OU=appadmin,O=Fluddi,L=Dhaka,ST=Dhaka,C=BD",
    roles: [
      { role : "clusterAdmin", db : "admin" },
      { role: "dbOwner", db: "fluddi" },
    ],
    writeConcern: { w: "majority" , wtimeout: 5000 }
  }
)
```

Validate
```
db.getSiblingDB("$external").auth(
  {
    mechanism: "MONGODB-X509",
    user: "emailAddress=support@fluddi.com,CN=*.fluddi.com,OU=appadmin,O=Fluddi,L=Dhaka,ST=Dhaka,C=BD"
  }
)
```