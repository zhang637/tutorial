#HBASE
##Hbase 测试
切换到hbase用户：

	su - hbase
进入hbase客户端命令行：
	
	hbase shell
查看数据库状态：

	status 'biaoming'
查看数据库表：
	
	！list
启动trift

	hbase thrift start -p 8001 --infoport <infoport>
启动rest服务
	
	hbase rest start -p 8000 --infoport <infoport>
后台方式启动trift服务
	
	hbase-daemon.sh start thrift -p 8001 --infoport <infoport>

##HBase 基础
	create 'traffic_hfile',{NAME => '201210', VERSIONS => 2}
	create ‘scores','grade', ‘course' 
	
	put ‘scores','Tom','grade:','5′ 
	put ‘scores','Tom','course:math','97′ 
	put ‘scores','Tom','course:art','87′ 
	put ‘scores','Jim','grade','4′ 
	put ‘scores','Jim','course:','89′ 
	put ‘scores','Jim','course:','80′ 
	
	get ‘t1′, ‘r1′, {COLUMN => ‘c1′} 
	scan 'docs', {COLUMNS => ['201210'], LIMIT => 1, STARTROW => '194273'}
	
	disable 't1'
	drop 't1'
	
	create 'test1', 'lf', 'sf'
				-- lf: column family of LONG values (binary value)
				-- sf: column family of STRING values
				-- 一个用户（userX），在什么时间（tsX），作为rowkey
				-- 对什么产品（value：skuXXX），做了什么操作作为列名，比如，c1: click from 	homepage; c2: click from ad; s1: search from homepage; b1: buy

	put 'test1', 'user1|ts1', 'sf:c1', 'sku1'
	put 'test1', 'user1|ts2', 'sf:c1', 'sku188'
	put 'test1', 'user1|ts3', 'sf:s1', 'sku123'
	put 'test1', 'user2|ts4', 'sf:c1', 'sku2'
	put 'test1', 'user2|ts5', 'sf:c2', 'sku288'
	put 'test1', 'user2|ts6', 'sf:s1', 'sku222'


	scan 'test1', FILTER=>"ValueFilter(=,'binary:sku188')"
	scan 'test1', FILTER=>"ValueFilter(=,'substring:188')"
	scan 'test1', FILTER=>"ValueFilter(=,'substring:88')"
	scan 'test1', FILTER=>"ColumnPrefixFilter('c2') AND ValueFilter(=,'substring:88')"
	scan 'test1', FILTER=>"ColumnPrefixFilter('s') AND ( ValueFilter(=,'substring:123') OR ValueFilter(=,'substring:222') )"
	scan 'test1', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(=,'binary:sku188') AND KeyOnlyFilter()"
	scan 'test1', FILTER => "PrefixFilter ('user1')"
	scan 'test1', {STARTROW=>'user1|ts2', FILTER => "PrefixFilter ('user1')"}
	scan 'test1', {STARTROW=>'user1|ts2', STOPROW=>'user2'}

##HBase 高级用法

-- hive hbase mapping --
CREATE EXTERNAL TABLE user_app_cookie_list ( username STRING, app1_cookie_id BIGINT, app2_cookie_id BIGINT )
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, lf:app1#b, lf:app2#b")
TBLPROPERTIES("hbase.table.name" = "test1");

select * from user_app_cookie_list;

-- hive hbase mapping cf with binary --

http://www.abcn.net/2013/11/hive-hbase-mapping-column-family-with-binary-value.html

CREATE EXTERNAL TABLE ts_string ( username STRING, visits map<string, int> )
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, cf:#s:b")
TBLPROPERTIES("hbase.table.name" = "test1");

CREATE EXTERNAL TABLE ts_int ( username STRING, visits map<int, int> )
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, cf:#s:b")
TBLPROPERTIES("hbase.table.name" = "test1");

CREATE EXTERNAL TABLE ts_int_long ( username STRING, visits map<int, bigint> )
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key, cf:#s:b")
TBLPROPERTIES("hbase.table.name" = "test1");

select * from ts_int
lateral view explode(visits) t as ts, page;

select username, ts, page_id from ts_int
lateral view explode(visits) t as ts, page_id;

select username, pos, ts, page_id from ts_int
lateral view posexplode(visits) t as pos, ts, page_id;

username   pos   ts             page_id
user1      1     1399999999     9
user1      2     1400000000     8
user1      3     1400000001     7
user1      4     1400000002     8443
user2      1     1500000000     17
user2      2     1500000001     8444

select username, from_unixtime(ts), page_id from ts_int lateral view explode(visits) t as ts, page_id;




## 开发环境配置
13/12/11 09:45:33 INFO client.ZooKeeperRegistry: ClusterId read in ZooKeeper is null

在resource目录，或者编译根目录，增加一个hbase-site.xml

```    
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hdp1:2181</value>
    </property>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hdp1:8020/apps/hbase/data</value>
    </property>
    <property>
        <name>zookeeper.znode.parent</name>
        <value>/hbase-unsecure</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
```
Maven工程配置

```
	<dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
    </dependency>
	<dependency>
			<groupId>org.apache.hbase</groupId>
			<artifactId>hbase-client</artifactId>
			<version>${hbase.version}</version>
	</dependency>
	<dependency>
			<groupId>org.apache.hbase</groupId>
			<artifactId>hbase-server</artifactId>
			<version>${hbase.version}</version>
	</dependency>
```
工程代码如下：

```
    static Configuration hbaseConfiguration = HBaseConfiguration.create();
    static {
        hbaseConfiguration.addResource("hbase-site.xml");
    }
    HBaseAdmin admin = new HBaseAdmin(hbaseConfiguration);
    if (admin.tableExists(tablename)) {// 如果表已经存在
        System.out.println(tablename + "表已经存在!----------");
    } else {
        TableName tableName = TableName.valueOf(tablename);
        HTableDescriptor tableDesc = new HTableDescriptor(tableName);
        tableDesc.addFamily(new HColumnDescriptor(columnFamily));
        admin.createTable(tableDesc);
        System.out.println(tablename + "表已经成功创建!----------------");
    }
```
##pheonix 安装测试
配置

```
<property>
    <name>hbase.defaults.for.version.skip</name>
    <value>true</value>
</property>
```
配置

```
hbase.regionserver.wal.codec : org.apache.hadoop.hbase.regionserver.wal.WALCellCodec  --> org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec
hbase.region.server.rpc.scheduler.factory.class : org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory
hbase.rpc.controllerfactory.class : org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
phoenix.functions.allowUserDefinedFunctions : true
```

```
cd /usr/hdp/current/phoenix-client/bin/ 
./psql.py localhost:2181:/hbase-unsecure /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.sql /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.csv /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT_QUERIES.sql
```

##HBase eclipse开发测试

##HBase 服务器端运行
