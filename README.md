flumeng-kafka-plugin
====================

Technical Specifications


Flume 1.4

Kafka 0.8.0 Beta
===============================================================
update by zqh on 2014-05-13

1. 修改pom.xml中kafka的版本从0.8.0-beta1升级到0.8.1
Kafaka 2.9.2_0.8.1
    Kafka: 0.8.1
    Scala: 2.9.2

2. 编译生成jar包:
$ cd flumeng-kafka-plugin
$ mvn package -DskipTests

maven-jar-plugin命令会在package下生成flumeng-kafka-plugin.jar
maven-dependency-plugin会在libs下生成kafka_2.9.2-0.8.1.jar等jar包

如果更改了pom.xml中某些包的版本，要重新进行编译!

3.将上面libs下的所有包和flumeng-kafka-plugin.jar拷贝到$FLUME_HOME/lib下
  flumeng-kafka-plugin.jar
  kafka_2.9.2-0.8.1.jar
  metrics-annotation-2.2.0.jar
  metrics-core-2.2.0.jar
  scala-compiler-2.9.2.jar
  scala-library-2.9.2.jar
  zkclient-0.1.jar

flume的版本为1.4.0. 官方的版本依赖的zookeeper为: 3.3.4. 而cdh4.6.0依赖的zk版本为3.4.5

4. 启动kafka
这里kafka的版本为: Kafka Version: 2.10-0.8.1 也没有关系.不过最好用kafka官方的2.9.2-0.8.1
其实上面libs下的一些包都能在$KAFKA_HOME/lib下找到
  kafka_2.9.2-0.8.1.jar
  metrics-annotation-2.2.0.jar
  metrics-core-2.2.0.jar
  scala-library-2.9.2.jar
  zkclient-0.3.jar
所以上面第三步除了flumeng-kafka-plugin.jar外的jar包也可以从$KAFKA_HOME/lib下拷贝到$FLUME_HOME/lib下

1) 启动zookeeper
$ bin/zookeeper-server-start.sh config/zookeeper.properties

2) 启动kafka server
$ bin/kafka-server-start.sh config/server.properties
#bin/kafka-server-start.sh config/server-1.properties
#bin/kafka-server-start.sh config/server-2.properties

3) 创建topic
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafkaToptic
注意这里的topic的名称和conf/flume-conf.properties下的topicName要对应
上面我们只启动了一个kafka server, 如果启动多个server, 则上面创建topic的参数replication-factor改成3

4) 创建consumer
$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic kafkaToptic

5) 创建producer (test if consumer work, u can type some word below, and check above)
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kafkaToptic
参数topic的名称都和创建topic时指定的名称一样. 这有这样才能publish-subscribe


5. 启动flume
flume和kafka集群的目的是: 数据经过flume,被flume收集后, 传给kafka消息队列. 简要流程:
a. flume的source通过tail -f file读取数据
b. 数据会被flume-kafka-plugin的KafkaSink收集数据写到topic=kafkaToptic中
c. KafkaSource从队列中读取topic=kafkaToptic的数据

1) 复制flumeng-kafka-plugin/conf/flume-conf.properties到$FLUME_HOME/conf下 并修改:
producer.sources.s.type = exec
producer.sources.s.command = tail -f ~/data/tail
注意: 如果kafka server进行集群, 则要修改producer.sinks.r.metadata.broker.list以逗号分割
如果zookeeper集群, 则修改consumer.sources.s.zookeeper.connect

2) 启动flume server
$ bin/flume-ng agent --conf conf --conf-file conf/flume-conf.properties --name producer -Dflume.root.logger=INFO,console

6. echo "Hello World" >> ~/data/tail

7. 在kafka-console-consumer能够看到追加的数据.
至此, 数据从flume流转到kafka的整合完毕.

Next Step: kafka --> Storm
