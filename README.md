#Fast Data Cluster

## Content

In case you need a local cluster providing Kafka, Cassandra and Spark you're at the right place.

* [Apache Kafka 1.1.0](http://kafka.apache.org/11/documentation.html)
* [Elastic Search 6.2.4](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/index.html)
* [Logstash 6.2.4](https://www.elastic.co/guide/en/logstash/6.2/index.html)
* [Kibana 6.2.4](https://www.elastic.co/guide/en/kibana/6.2/index.html)
* [Grafana 5.1.2](https://grafana.com)
* [Graphite](https://graphiteapp.org)
* [Jmxtrans Agent 1.2.6](https://github.com/jmxtrans/jmxtrans-agent/)

## Prerequisites

* [Vagrant](https://www.vagrantup.com) (tested with 2.1.1)
* [VirtualBox](http://virtualbox.org) (tested with 5.2.12)
* [Ansible](http://docs.ansible.com/ansible/index.html) (tested with 2.5.2)
* The VMs take approx 10 GB of RAM, so you should have more than that.


:warning: Vagrant might ask you for your admin password. The reason behind is, that `vagrant-hostsupdater` is used to have the vms available with their names in your network.

## Init

```bash
git clone https://github.com/markush81/kafka-cluster.git
vagrant up
```

## Cluster

The result if everything wents fine should be

![Kafka Cluster](doc/kafka-cluster.png)


## Coordinates

#### Servers

| IP | Hostname | Description | Settings |
|:--- |:-- |:-- |:-- |
|192.168.10.2|mon-1|running elk, grafana, graphite and metricbeat| 4096 MB RAM |
|192.168.10.3|kafka-1|running a kafka broker and metricbeat| 2048 MB RAM |
|192.168.10.4|kafka-2|running a kafka broker and metricbeat| 2048 MB RAM |
|192.168.10.5|kafka-3|running a kafka broker and metricbeat| 2048 MB RAM |


### Connections

| Name |  |
|:-- |:-- |
|Zookeeper|kafka-1:2181,kafka-2:2181,kafka-3:2181|
|Kafka Brokers|kafka-1:9092,kafka-2:9092,kafka-3:9092|
|Kibana|[http://mon-1:5601](http://mon-1:5601)|
|Elasticsearch|[http://mon-1:9200](http://mon-1:9200)|
|Grafana|[http://mon-1:3000](http://mon-1:3000)|
|Graphite|[http://mon-1](http://mon-1)|


# Monitoring

## System Overview

![System Overview Part 1](doc/system_overview_1.png)
![System Overview Part 2](doc/system_overview_2.png)

## Kafka Overview

![Kafka Overview](doc/kafka_overview.png)

:warning: still in progess to add more!

# Usage

## Zookeeper

```bash
lucky:~ markus$ vagrant ssh kafka-1
[vagrant@kafka-1 ~]$ zkCli.sh -server kafka-1:2181/
Connecting to kafka-1:2181/
...

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: zookeeper-1:2181,zookeeper-3:2181(CONNECTED) 0] ls /
[cluster, controller, controller_epoch, brokers, zookeeper, admin, isr_change_notification, consumers, config]
[zk: zookeeper-1:2181,zookeeper-3:2181(CONNECTED) 1]

```

## Kafka

### Topic Creation

```bash
lucky:~ markus$ vagrant ssh kafka-1
[vagrant@kafka-1 ~]$ kafka-topics.sh --create --zookeeper kafka-1:2181 --replication-factor 2 --partitions 6 --topic sample
Created topic "sample".
[vagrant@kafka-1 ~]$ kafka-topics.sh --zookeeper kafka-1 --topic sample --describe
Topic:sample	PartitionCount:6	ReplicationFactor:2	Configs:
	Topic: sample	Partition: 0	Leader: 1	Replicas: 1,2	Isr: 1,2
	Topic: sample	Partition: 1	Leader: 2	Replicas: 2,3	Isr: 2,3
	Topic: sample	Partition: 2	Leader: 3	Replicas: 3,1	Isr: 3,1
	Topic: sample	Partition: 3	Leader: 1	Replicas: 1,3	Isr: 1,3
	Topic: sample	Partition: 4	Leader: 2	Replicas: 2,1	Isr: 2,1
	Topic: sample	Partition: 5	Leader: 3	Replicas: 3,2	Isr: 3,2
[vagrant@kafka-1 ~]$
```
### Producer

```bash
[vagrant@kafka-1 ~]$ kafka-console-producer.sh --broker-list kafka-1:9092,kafka-3:9092 --topic sample
[2017-04-22 15:27:41,035] WARN Removing server kafka-1::9092 from bootstrap.servers as DNS resolution failed for kafka-1: (org.apache.kafka.clients.ClientUtils)
Hey, is Kafka up and running?
```

### Consumer

```bash
[vagrant@kafka-1 ~]$ kafka-console-consumer.sh --bootstrap-server kafka-1:9092,kafka-3:9092 --topic sample --from-beginning
Hey, is Kafka up and running?
```

### Producer Perf Test

```bash
[vagrant@kafka-1 ~]$ kafka-producer-perf-test.sh --producer-props bootstrap.servers="kafka-1:9092,kafka-2:9092,kafka-3:9092" --topic sample --num-records 2000 --throughput 100 --record-size 256

```
