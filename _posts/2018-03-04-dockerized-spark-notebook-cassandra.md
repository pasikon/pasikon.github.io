---
title: "Dockerized Spark Notebook with Cassandra"
excerpt: "How to start with all-dockerized Data Science environment"
last_modified_at: 2018-03-04T17:10:58-05:00
header:
  teaser: "assets/images/spark-logo.png"
tags: 
  - scala
  - spark
  - cassandra
  - docker
toc: true
---

If you are in need of creating environment to do some Data Science, experiment or learn Spark, Docker is the way to do this within couple of minutes. I hope instructions below will speed up the process even further!

### Assemble Spark Notebook distribution

First thing you do is simply go to http://spark-notebook.io/ and request building of Docker image that meets your requirements, I have chosen like this:

![no-alignment](assets/images/screenshot-spark-notebook.png)

You also put your email there below in order ot be informed that build has completed & receive links to it.

NOTE: in my case the Docker image name was wrongly specified, snapshot version was incorect. If you are building from master you need to paste correct snapshot version there.

Ok so we got email containing something like this:
 
```bash
docker pull andypetrella/spark-notebook:0.7.0-SNAPSHOT-scala-2.11.8-spark-2.2.1-hadoop-2.7.2-with-hive
docker run -p 9001:9001 andypetrella/spark-notebook:0.7.0-SNAPSHOT-scala-2.11.8-spark-2.2.1-hadoop-2.7.2-with-hive
```

Correct snapshot verion at the moment was `0.9.0-SNAPSHOT` so these commands won't even work, we want also to tune our setup a bit so don't issue any commands like this right now!
Correct pull in my case looked like this:

```bash
docker pull andypetrella/spark-notebook:0.9.0-SNAPSHOT-scala-2.11.8-spark-2.2.1-hadoop-2.7.2-with-hive
```

### Create Docker network

We want to simplify Spark Notebook container interaction with other containers, lets put them all in same network. That way you can reference other containers hosts via containers name without any mappings in `/etc/hosts` or whatever! 

Create `lizard` network:

```bash
docker network create --driver=bridge --subnet=192.168.0.0/16 lizard
```

### Create all required containers

Let's start all stuff, all in the same `lizard` network. Docker command argument are let's say basic ones, really easy to find the meaning with Docker Documentation :)

Please notice that we are specifying names for all containers to reference them later!

#### Cassandra:

```bash
docker run --net lizard --name liz-cass -d -e CASSANDRA_BROADCAST_ADDRESS=0.0.0.0 -p 7000:7000 -p 7199:7199 -p 9042:9042 -p 9160:9160 cassandra:latest
```

#### Spark Notebook:

```bash
docker run -d --net lizard --name sp_notebook -p 9000:9000 -p 4040-4045:4040-4045 -v v:/shared:/win_shared andypetrella/spark-notebook:0.9.0-SNAPSHOT-scala-2.11.8-spark-2.2.1-hadoop-2.7.2-with-hive
```

### Playing around with notebook!

Now navigate to http://localhost:9000/, you will see directory structure with example notebooks provided with Spark Notebook.

Open `cassandra/Getting Started with Spark and Cassandra` notebook.

First 4 cells in the notebook about setting up cassandra-connector dependecies & other configuration are obsolete, to confugure up our dockerized Cassandra instance do the following:

Click `Edit/Edit Notebook Metadata` menu and paste there updated config:

```json
"customLocalRepo": "/tmp/repo",
"customdeps": [
  "com.datastax.spark % spark-cassandra-connector_2.11 % 2.0.7"
],
"customSparkConf" : {
  "spark.cassandra.connection.host": "liz-cass"
}
```

I have updated connector to newest available version, also please note that we can reference to Cassandra per container name!

After that restart kernel by `Kernel/Restart` menu, open browser console (F12 ?) to see new dependency being downloaded, also if theres any error it will be displyed there.

Go to notebook section `Create a keyspace and a table for our test` and you should be able to run all the examples contining creating some schema in Cassandra and writing RDD data there.

