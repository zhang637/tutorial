# Flume 
## 配置
source是：exec


```
ht001.sources = s1
ht001.sinks = k1
ht001.channels = c1

ht001.sources.s1.type = exec
ht001.sources.s1.command = tail -F /opt/fluminputedata

ht001.channels.c1.type = memory
ht001.channels.c1.capacity = 1000
ht001.channels.c1.transactionCapacity = 100

ht001.sinks.k1.type = logger

ht001.sources.s1.channels = c1
ht001.sinks.k1.channel = c1
```

数据源：spooldir

目标：kafka

```
[root@hadoop4 flume]# cat conf/kafka_zyl.conf 
producer.sources = s
producer.sinks = r
producer.channels = c
producer.sinks.r.channel = c
producer.sources.s.channels = c

#producer.sources.s.type = exec
#producer.sources.s.command = tail -F /opt/oneM/

producer.sources.s.type = spooldir
producer.sources.s.spoolDir = /opt/oneM/
producer.sources.s.fileHeader = true
producer.sources.s.batchSize = 100

producer.channels.c.type = memory
producer.channels.c.capacity = 1000
producer.channels.c.transactionCapacity = 100

producer.sinks.r.type = org.apache.flume.sink.kafka.KafkaSink
producer.sinks.r.brokerList=hadoop4:6667
producer.sinks.r.partition.key=0
producer.sinks.r.partitioner.class=org.apache.flume.plugins.SinglePartition
producer.sinks.r.serializer.class=kafka.serializer.StringEncoder
producer.sinks.r.request.required.acks=0
producer.sinks.r.max.message.size=1000000
producer.sinks.r.producer.type=sync
producer.sinks.r.custom.encoding=UTF-8
producer.sinks.r.topic=test_fk1
```
数据源：syslog命令
```
ht002.sources = s1ht002.channels = c1ht002.sinks = k1ht002.sources.s1.type = syslogtcpht002.sources.s1.port = 4321ht002.sources.s1.host = hadoop01ht002.channels.c1.type = memoryht002.channels.c1.capacity = 1000ht002.channels.c1.transactionCapacity = 100ht002.sinks.k1.type = hdfsht002.sinks.k1.hdfs.path = hdfs://hadoop01:8020/miras/flumeht002.sinks.k1.hdfs.filePrefix = Sysloght002.sinks.k1.hdfs.round = trueht002.sinks.k1.hdfs.roundValue = 10ht002.sinks.k1.hdfs.roundUnit = minuteht002.sinks.k1.channel = c1ht002.sources.s1.channels = c1
```
# 模拟数据
创建一个shell脚本

```
for i in {1..10000}
 do
 echo "exec  tail$i" >> /opt/fluminputedata
 done
```

## 运行
	cd /opt/taxiJar_test/
	java -jar input.jar 
	bin/kafka-console-consumer.sh --zookeeper hadoop2:2181,hadoop3:2181,hadoop4:2181 --topic test_fk --from-beginning
	bin/flume-ng agent --conf conf --conf-file conf/zyl.conf --name producer -Dflume.root.logger=INFO,console
	flume-ng agent -c conf -f conf/exec_tails.conf -n ht001 -Dflume.root.logger=INFO,console
## 运行2
	mkdir /HDFS/miras
	mkdir /HDFS/miras/flume
	bin/flume-ng agent -c conf -f conf/ht_hdfs.conf -n ht002 -Dflume.root.logger=INFO,console
	echo "aaaaaaaaaaabbbbbbbbbbbccccccccccccddddddddddd" | nc test 4321
	
