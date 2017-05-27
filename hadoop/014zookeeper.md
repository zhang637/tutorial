# zookeeper 
## 路径

	bin/zookeeper-client -server node2,node3,node4 <<EOF
	ls /brokers/topics
	quit
	EOF

查看数据状态
	
	stat /brokers/topics/mypart6
	
	cZxid = 0x5001d64e7
	ctime = Thu Apr 27 11:42:38 CST 2017
	mZxid = 0x5001d64e7
	mtime = Thu Apr 27 11:42:38 CST 2017
	pZxid = 0x5001d64eb
	cversion = 1
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 94
	numChildren = 1
命令行操作
bin/zkCli.sh -127.0.0.1:2181
	
```
[zk: localhost:2181(CONNECTED) 6] create /zktest mydata
Created /zktest
[zk: localhost:2181(CONNECTED) 12] ls /
[zktest, zookeeper]
[zk: localhost:2181(CONNECTED) 7] ls /zktest
[]
[zk: localhost:2181(CONNECTED) 13] get /zktest
mydata
cZxid = 0x1c
ctime = Thu Jun 11 10:58:06 CST 2015
mZxid = 0x1c
mtime = Thu Jun 11 10:58:06 CST 2015
pZxid = 0x1c
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
[zk: localhost:2181(CONNECTED) 14] set /zktest junk
cZxid = 0x1c
ctime = Thu Jun 11 10:58:06 CST 2015
mZxid = 0x1f
mtime = Thu Jun 11 10:59:08 CST 2015
pZxid = 0x1c
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: localhost:2181(CONNECTED) 15] delete /zktest
[zk: localhost:2181(CONNECTED) 16] ls /
[zookeeper]
```
##Kafka偏移量查看
```
[root@node2 kafka]# bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list node1:6667,node2:6667,node3:6667,node4:6667 -topic zk_log --time -1
{metadata.broker.list=node1:6667,node2:6667,node3:6667,node4:6667, request.timeout.ms=1000, client.id=GetOffsetShell, security.protocol=PLAINTEXT}
zk_log:2:37980
zk_log:1:37980
zk_log:3:37979
zk_log:0:37980
```

```
public static void main(String[] args) {
　　　　　　String zkConnect = "11.11.184.162:2181,11.11.184.165:2181,11.11.184.178:2181";
          int sessionTimeout = 5000;  
          int connectionTimeout = 5000; 
          ZkClient zkClient = new ZkClient(zkConnect, sessionTimeout, connectionTimeout,new ZkSerializerSuper());
          //修改指定消费组在zookeeper中的偏移量
　　　　　　map.put("0","12345");
　　　　　　map.put(2,"23451");
　　　　　　map.put(1,"23541");
          setPersistentPathInZk(map,zkClient);//map存放的是每个分区对应的偏移量
}

//调用ZKUtils中的updatePersistentPath方法进行修改
private static void setPersistentPathInZk(Map map, ZkClient zkClient) {
　　　　String group_id = "kafkawarning01";
　　　　String topic = "topic1";
　　　　Set set = map.keySet();
        for (Object key : set) {
            ZkUtils.updatePersistentPath(zkClient, "/consumers/"+group_id+"/offsets/"+topic+"/"+key, map.get(key).toString());
        }
        
    }
```
