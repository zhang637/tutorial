# HBASE
## HBase 测试
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

## HBase 基础
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

## HBase2Hive
创建hbase表并增加数据
	
	create 'short_urls', {NAME=>'u'},{NAME=>'s'}
	put 'short_urls', 'bit.ly/aaaa', 's:hits', '100'
	put 'short_urls', 'bit.ly/aaaa', 'u:url', 'hbase.apache.org'
	put 'short_urls', 'bit.ly/abcd', 's:hits', '123'
	put 'short_urls', 'bit.ly/abcd', 'u:url', 'example.com/foo'
	scan 'short_urls'
创建hive表
	
	CREATE EXTERNAL TABLE traffic_mr(key string, values string) 	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
	WITH SERDEPROPERTIES("hbase.columns.mapping" = ":key,201210:content") 	TBLPROPERTIES("hbase.table.name"="traffic_mr");
	
## HBase2Hive进阶
hive中map的key将作为hbase的列名，map的value将作为该列名对应的值
	
	CREATE EXTERNAL TABLE traffic_hfile(value map<string,String>, row_key int) 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = "201210:,:key")
	;

hive 0.12开始支持，使用通配符 * 来获取多个列。例如：想获取以某一前缀开始的列，创建表如下：
	
	CREATE TABLE hbase_table_1(value map<string,int>, row_key int) 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = "cf:col_prefix.*,:key");
hive 的数据类型处理，如：二进制
	
	CREATE TABLE hbase_table_1 (key int, value string, foobar double)
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key#b,cf:val,cf:foo#b");

	CREATE TABLE hbase_table_1 (key int, value string, foobar double)
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES (
	"hbase.columns.mapping" = ":key,cf:val#s,cf:foo",
	"hbase.table.default.storage.type" = "binary");
	
将hbase的复合行键映射为hive的struct结构体
	
	CREATE EXTERNAL TABLE delimited_example(key struct<f1:string, f2:string>, value string) 
	ROW FORMAT DELIMITED 
	COLLECTION ITEMS TERMINATED BY '~' 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
	WITH SERDEPROPERTIES ('hbase.columns.mapping'=':key,f:c1');

	CREATE TABLE hbase_ck_4(key struct<col1:string,col2:string,col3:string>, value string)
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES (
	"hbase.table.name" = "hbase_custom2",
	"hbase.mapred.output.outputtable" = "hbase_custom2",
	"hbase.columns.mapping" = ":key,cf:string",
	"hbase.composite.key.factory"="org.apache.hadoop.hive.hbase.SampleHBaseKeyFactory2");
	
	"hbase.composite.key.factory" should be the fully qualified class name of a class implementing HBaseKeyFactory. 
	See SampleHBaseKeyFactory2 for a fixed length example in the same package. This class must be on your classpath in order for the above example to work

Avro格式数据存储在hbase中

	CREATE EXTERNAL TABLE test_hbase_avro
	ROW FORMAT SERDE 'org.apache.hadoop.hive.hbase.HBaseSerDe'
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES (
	"hbase.columns.mapping" = ":key,test_col_fam:test_col",
	"test_col_fam.test_col.serialization.type" = "avro",
	"test_col_fam.test_col.avro.schema.url" = "hdfs://testcluster/tmp/schema.avsc")
	TBLPROPERTIES (
	"hbase.table.name" = "hbase_avro_table",
	"hbase.mapred.output.outputtable" = "hbase_avro_table",
	"hbase.struct.autogenerate"="true");

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
