# 创建主题
	bin/kafka-topics.sh --create --zookeeper hadoop2:2181,hadoop3:2181,hadoop4:2181 --replication-factor 1 --partitions 1 --topic test_fk1

# 主题查看
	bin/kafka-topics.sh --list --zookeeper hadoop1:2181,hadoop2:2181,hadoop3:2181
# 生产者
	bin/kafka-console-producer.sh --broker-list localhost:6667 --topic test_fk1
# 消费者
	bin/kafka-console-consumer.sh --zookeeper hadoop2:2181,hadoop3:2181,hadoop1:2181 --topic test_fk1 --from-beginning
# 话题删除
	bin/kafka-topics.sh  --delete --zookeeper node2:2181,node3:2181,node4:2181 --topic zk_log
	bin/kafka-topics.sh --list --zookeeper node2:2181,node3:2181,node4:2181
	bin/zookeeper-client -server node2,node3,node4
	ls /brokers/topics
	rmr /brokers/topics/xxx

另外被标记为marked for deletion的topic你可以在zookeeper客户端中通过命令获得：ls /admin/delete_topics/【topic name】，如果你删除了此处的topic，那么marked for deletion 标记消失zookeeper 的config中也有有关topic的信息： ls /config/topics
## 修改消费组的偏移量


# KAFKA监听工具部署
```
#!/bin/bash
java -cp KafkaOffsetMonitor-assembly-0.2.0.jar \
com.quantifind.kafka.offsetapp.OffsetGetterWeb \
--zk hadoop1:2181,hadoop2:2181,hadoop3:2181 \
--port 8089 \
--refresh 10.seconds \
--retain 2.days
```

#kafka高级配置
##时效配置
参数  		 | 说明
------------|---------
broker.id =0 |每一个broker在集群中的唯一表示，要求是正数。*当该服务器的IP地址发生改变时*，broker.id没有变化，则不会影响consumers的消息情况
log.dirs=/data/kafka-logs |kafka数据的存放地址，多个地址,用逗号分割/data/kafka-logs-1，/data/kafka-logs-2
port =9092 | broker   server服务端口
message.max.bytes=6525000 | 表示消息体的最大大小，单位是字节
num.network.threads=4 | broker处理消息的最大线程数，一般情况下不需要去修改
num.io.threads=8 | broker处理磁盘IO的线程数，数值应该大于你的硬盘数
background.threads=4 | 一些后台任务处理的线程数，例如过期消息文件的删除等，一般情况下不需要去做修改
queued.max.requests=500 | 等待IO线程处理的请求队列最大数，若是等待IO的请求超过这个数值，那么会停止接受外部消息，应该是一种自我保护机制。
host.name | broker的主机地址，若是设置了，那么会绑定到这个地址上，若是没有，会绑定到所有的接口上，并将其中之一发送到ZK，一般不设置
socket.send.buffer.bytes=102400 | socket的发送缓冲区，socket的调优参数SO_SNDBUFF
socket.receive.buffer.bytes=102400 | socket的接受缓冲区，socket的调优参数SO_RCVBUFF
socket.request.max.bytes=100.1024.1024 | socket请求的最大数值，防止serverOOM,message.max.bytes必然要小于socket.request.max.bytes，会被topic创建时的指定参数覆盖
log.segment.bytes=1024.1024.1024 | topic的分区是以一堆segment文件存储的，这个控制每个segment的大小，会被topic创建时的指定参数覆盖
log.roll.hours =24.7 | 这个参数会在日志segment没有达到log.segment.bytes设置的大小，也会强制新建一个segment。会被topic创建时的指定参数覆盖
log.cleanup.policy   = delete | 日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被topic创建时的指定参数覆盖
log.retention.minutes=3days | 数据存储的最大时间超过这个时间会根据log.cleanup.policy设置的策略处理数据，也就是消费端能够多久去消费数据log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖
log.retention.bytes=-1 | topic每个分区的最大文件大小，一个topic的大小限制=分区数*log.retention.bytes。-1没有大小限log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除，会被topic创建时的指定参数覆盖

##KAFKA与Zookeeper关系
kafka使用zookeeper来存储一些meta信息,并使用了zookeeper watch机制来发现meta信息的变更并作出相应的动作(比如consumer失效,触发负载均衡等)


* Broker node registry: 当一个kafka broker启动后,首先会向zookeeper注册自己的节点信息(临时znode),同时当broker和zookeeper断开连接时,此znode也会被删除.格式: /broker/ids/[0...N]   -->host:port;其中[0..N]表示broker id,每个broker的配置文件中都需要指定一个数字类型的id(全局不可重复),znode的值为此broker的host:port信息.
* Broker Topic Registry: 当一个broker启动时,会向zookeeper注册自己持有的topic和partitions信息,仍然是一个临时znode.格式: /broker/topics/[topic]/[0...N]  其中[0..N]表示partition索引号.
* Consumer and Consumer group: 每个consumer客户端被创建时,会向zookeeper注册自己的信息;此作用主要是为了"负载均衡".一个group中的多个consumer可以交错的消费一个topic的所有partitions;简而言之,保证此topic的所有partitions都能被此group所消费,且消费时为了性能考虑,让partition相对均衡的分散到每个consumer上.
* Consumer id Registry: 每个consumer都有一个唯一的ID(host:uuid,可以通过配置文件指定,也可以由系统生成),此id用来标记消费者信息.格式: /consumers/[group_id]/ids/[consumer_id]仍然是一个临时的znode,此节点的值为{"topic_name":#streams...},即表示此consumer目前所消费的topic + partitions列表.
* Consumer offset Tracking: 用来跟踪每个consumer目前所消费的partition中最大的offset.格式: /consumers/[group_id]/offsets/[topic]/[broker_id-partition_id]-->offset_value
此znode为持久节点,可以看出offset跟group_id有关,以表明当group中一个消费者失效,其他consumer可以继续消费.
* Partition Owner registry: 用来标记partition被哪个consumer消费.临时znode格式: /consumers/[group_id]/owners/[topic]/[broker_id-partition_id] -->consumer_node_id当consumer启动时,所触发的操作:A) 首先进行"Consumer id Registry";B) 然后在"Consumer id Registry"节点下注册一个watch用来监听当前group中其他consumer的"leave"和"join";只要此znode path下节点列表变更,都会触发此group下consumer的负载均衡.(比如一个consumer失效,那么其他consumer接管partitions). C) 在"Broker id registry"节点下,注册一个watch用来监听broker的存活情况;如果broker列表变更,将会触发所有的groups下的consumer重新balance.
* Producer端使用zookeeper用来"发现"broker列表,以及和Topic下每个partition leader建立socket连接并发送消息.
* Broker端使用zookeeper用来注册broker信息,已经监测partition leader存活性.
* Consumer端使用zookeeper用来注册consumer信息,其中包括consumer消费的partition列表等,同时也用来发现broker列表,并和partition leader建立socket连接,并获取消息.

##KAFKA OFFSET
	bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list node2:6667,node3:6667,node4:6667  --time -1 --topic 




