# ZooKeeper

## What is ZooKeeper?

[ZooKeeper](https://zookeeper.apache.org) is a reliable distributed service for coordination, which help nodes share consistent configurations, metadata and status. 

Most Important Functions:

- Configuration Management
- Metadata storage
- Event Listenser
- Leader Election



## Quick Start

Precondition: a linux machine with jdk available 

Download Page: [Apache ZooKeeper](https://zookeeper.apache.org/releases.html)

First, start a stand-alone zookeeper service

```sh
tar -xzvf apache-zookeeper-x.y.z-bin.tar.gz

# Enter the root directory
cd apache-zookeeper-x.y.z-bin

# Create a config file
# Default configuration: 
# tickTime=2000
# dataDir=/tmp/zookeeper
# clientPort=2181
cp conf/zoo_sample.cfg conf/zoo.cfg

# Start Server
# If success, we will see:
# ZooKeeper JMX enabled by default
# Using config: zookeeper-path/bin/../conf/zoo.cfg
# Starting zookeeper ... STARTED
bin/zkServer.sh start

# Check server status
# If server works well, we will see:
# Using config: zookeeper-path/bin/../conf/zoo.cfg
# Client port found: 2181. Client address: localhost. Client SSL: false.
# Mode: standalone
bin/zkServer.sh status


# Stop server
# 
bin/zkServer.sh stop

```

Second, read or write some data(persistent node)

```sh
# show znodes of a path
ls /

# create a znode
create /node1 "hello"

# read node data
get /node1

# set node data
set /node1 "hello,zk"

# delete node data 
deleteall /node1
```



## ZNode

Zookeeper maintains a tree directory data structure similar to file system.

For example:

```
/
├── app
│   ├── config
│   ├── leader
│   └── workers
└── locks

```

**Znode Data**:

- Data in bytes (no more than 1MB)
- Child nodes (not ephemeral)
- Stat data (version, timestamp)

**Znode Type**:

Persistent Znode exists even if restarted or session lost, which is designed for configuration.

```sh
create /app/config "db=localhost"
```

Ephemeral Znode disappear as session lost,  (There is mo children Znode in Ephemeral Znode) which is designed for service registration/heart beat/online status.

```sh
create -e /workers/worker-A "10.0.0.1"
```

Persistent Sequential Znode is used for distributed lock.

``` sh
create -s /queue/job "task1"
```

result:

```
/queue
  └── job0000000001   (Persistent Sequential)
```

Ephemeral Sequential Znode  is used for distributed lock or leader election.

```sh
create -e -s /lock/node "A"
```

More than one client create ephemeral sequential znode in the same path. Client with the smallest sequential number gets the lock or wins the election.

