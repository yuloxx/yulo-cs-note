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

## Watcher

Watcher provide a light weight mechanism to notify data change. Watching events include:

- children znode change
- znode data change
- znode delete

Test:

Open a connection by zkCli.sh in session S1.

```sh
zkCli.sh -server localhost:2181
create /watch_example "initial_data"
```

Open another connection by zkCli.sh in session S2.

```sh
zkCli.sh -server localhost:2181
get -w /watch_example
```

Update the data in session S1

```sh 
set /watch_example "updated_data"
```

output in S2:

```
[zk: localhost:2181(CONNECTED) 0] get -w /watch_example
initial_data
[zk: localhost:2181(CONNECTED) 1]
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/watch_example zxid: 4

```

Notice:

Watcher takes effects **only once**. Once triggered, following events will not be notified.  To keep watch a znode, it's necessary to register again.



## Leader Election

Zookeeper provide a reliable and simple way for leader election: create ephemeral sequential and elect the instance with least sequence number.

Every instance of an application attempt to  create sequential znode under the election directory. The instance with the least sequential number is elected as the leader. 

```
/myapp/leader/instance-0000000001
/myapp/leader/instance-0000000002
/myapp/leader/instance-0000000003
...
```

Every instance watch the election znode directory. As the leader fails, sequential znode is removed, then election is triggered.



Example Project:

This project shows how to start a LeaderElection thread and work with business thread through a global state service. Maven package this project and run jar package to observe leader election process! 

```
+---main
|   +---java
|   |   \---org
|   |       \---example
|   |               Application.java
|   |               CuratorConfig.java
|   |               LeaderController.java
|   |               LeaderElectionService.java
|   |               LeaderStateService.java
|   |
|   \---resources
|           application.yml
|
pom.xml
```

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>my-spring</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring-boot.version>2.7.6</spring-boot.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.3</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

application.yml:

```yml
zookeeper:
  connect-string: "a.b.c.d:xxxx"

```

Application

```java
package org.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

CuratorConfig

```java
package org.example;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CuratorConfig {

    @Value("${zookeeper.connect-string}")
    private String connectString;

    @Bean(initMethod = "start")
    public CuratorFramework curatorFramework() {
        return CuratorFrameworkFactory.builder()
                .connectString(connectString)
                .sessionTimeoutMs(60000)
                .connectionTimeoutMs(15000)
                .retryPolicy(new ExponentialBackoffRetry(1000, 3))
                .build();
    }

}

```

LeaderElectionService

```java
package org.example;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import org.apache.curator.framework.recipes.leader.LeaderSelectorListenerAdapter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.annotation.Resource;

@Service
public class LeaderElectionService {

    private static final String LEADER_PATH = "/demo-app/leader_election";

    private static final Logger log = LoggerFactory.getLogger(LeaderElectionService.class);

    @Resource
    private CuratorFramework client;

    @Resource
    private LeaderStateService leaderStateService;

    private LeaderSelector leaderSelector;

    @PostConstruct
    public void start() {
        leaderSelector = new LeaderSelector(client, LEADER_PATH, new LeaderSelectorListenerAdapter() {

            @Override
            public void takeLeadership(CuratorFramework curatorFramework) throws Exception {
                log.info("I'm leader now");
                leaderStateService.setLeader(true);

                try {
                    Thread.sleep(Long.MAX_VALUE);
                } finally {
                    leaderStateService.setLeader(false);
                    log.info("I am no longer Leader.");
                }
            }
        });
        leaderSelector.autoRequeue();
        leaderSelector.start();
        log.info("Leader election started.");
    }

    @PreDestroy
    public void stop() {
        if (leaderSelector != null) {
            leaderSelector.close();
        }
    }

}

```



LeaderStateService

```java
package org.example;

import org.springframework.stereotype.Component;

import java.util.concurrent.atomic.AtomicBoolean;

@Component
public class LeaderStateService {
    private final AtomicBoolean isLeader = new AtomicBoolean(false);

    public boolean isLeader() {
        return isLeader.get();
    }

    public void setLeader(boolean leader) {
        isLeader.set(leader);
    }
}

```



LeaderController

```java
package org.example;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class LeaderController {

    @Resource
    private LeaderStateService leaderStateService;

    @GetMapping("/isLeader")
    public String isLeader() {
        return leaderStateService.isLeader() ? "YES, I am leader" : "NO, I am follower";
    }
}

```



