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
创建hbase表并增加数据
	
	create 'short_urls', {NAME=>'u'},{NAME=>'s'}
	put 'short_urls', 'bit.ly/aaaa', 's:hits', '100'
	put 'short_urls', 'bit.ly/aaaa', 'u:url', 'hbase.apache.org'
	put 'short_urls', 'bit.ly/abcd', 's:hits', '123'
	put 'short_urls', 'bit.ly/abcd', 'u:url', 'example.com/foo'
	scan 'short_urls'
	
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

## HBase2Hive
创建hive表
	
	CREATE EXTERNAL TABLE traffic_mr(key string, values string) 	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
	WITH SERDEPROPERTIES("hbase.columns.mapping" = ":key,201210:content") 	TBLPROPERTIES("hbase.table.name"="traffic_mr");

## HBase2Hive进阶
hive中map的key将作为hbase的列名，map的value将作为该列名对应的值
	
	CREATE EXTERNAL TABLE traffic_hfile(value map<string,String>, row_key int) 
	STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
	WITH SERDEPROPERTIES ("hbase.columns.mapping" = "201210:,:key");

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
## pheonix 安装测试
修改配置

```
hbase.defaults.for.version.skip : true
hbase.regionserver.wal.codec : org.apache.hadoop.hbase.regionserver.wal.WALCellCodec  --> org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec
```
增加配置

```
hbase.region.server.rpc.scheduler.factory.class : org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory
hbase.rpc.controllerfactory.class : org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
phoenix.functions.allowUserDefinedFunctions : true
```
数据库结构语句

```
create INDEX APPTRAFFIC_INDEX11 on APP_TRAFFIC (USER_LON,USER_LAT,PACKAGE_NAME,START_TIME,END_TIME,NETWORK_NAME,NETWORK_TYPE) INCLUDE(UPLOAD_TRAFFIC,DOWNLOAD_TRAFFIC);

create INDEX APPTRAFFIC_INDEX22 on APP_TRAFFIC (ALLMOBILETRAFFIC,USER_LON,USER_LAT,COMPANYMODEL,START_TIME,END_TIME,NETWORK_NAME,NETWORK_TYPE) INCLUDE(UPLOAD_TRAFFIC,DOWNLOAD_TRAFFIC);

create INDEX APPTRAFFIC_INDEX33 on APP_TRAFFIC (ALLMOBILETRAFFIC,USER_LON,USER_LAT,START_TIME,END_TIME,NETWORK_NAME,NETWORK_TYPE) INCLUDE(UPLOAD_TRAFFIC,DOWNLOAD_TRAFFIC,OS,OS_VERSION,OS_ANDVERSION);

create INDEX APPTRAFFIC_INDEX44 on APP_TRAFFIC (ALLMOBILETRAFFIC,USER_LON,USER_LAT,START_TIME,END_TIME,NETWORK_NAME,NETWORK_TYPE) INCLUDE(UPLOAD_TRAFFIC,DOWNLOAD_TRAFFIC,PACKAGE_NAME,APP_NAME);

create index NWQUALITY_Index on NWQUALITY (GPSLAT,GPSLON,DAYTIME,NWOPERATOR,NWTYPE) INCLUDE(DLSPEED,ULSPEED,LATENCY);

create INDEX  SignalStrength_index1 ON Signal_Strength (
user_lon,
user_lat,
time_index,
rssi,
network_name,
network_type
);

create index signalstrength_Index3 on signal_strength (time_index DESC,network_name,network_type,rssi,landmark);

create table if not exists app_traffic(
id bigint not null,
date varchar(100),
network_name varchar(100),
user_lon varchar(100),
user_lat varchar(100),
start_time varchar(100),
end_time varchar(100),
mobile_type varchar(100),
network_type  varchar(100) ,
upload_traffic double,
download_traffic double,
package_name varchar(100),
app_name varchar(100),
cell_id varchar(100),
imei varchar(100),
os varchar(100),
os_version varchar(100),
os_andversion varchar(100),
companyModel varchar(100),
province varchar(100),
city varchar(100),
Landmark varchar(100),
allMobileTraffic  varchar(100)
CONSTRAINT strength_pk PRIMARY KEY(id));

create table if not exists NWQuality(
id bigint not null,
dayTime bigint,
NWOperator varchar(100),
tCount integer,
DLSpeed double,
ULSpeed double,
Latency double,
GpsLat varchar(100),
GpsLon varchar(100),
Province varchar(100),
City varchar(100),
Imei varchar(100),
NWType varchar(100),
mobileType varchar(100),
DeviceOS varchar(100),
DeviceVersion varchar(100),
DeviceOsVersion varchar(100),
company varchar(100),
model varchar(100),
CompanyModel varchar(100),
Landmark varchar(100)
CONSTRAINT nwQuality_pk PRIMARY KEY(id));

create table if not exists Signal_Strength(
id bigint not null,
time_index bigint,
network_name varchar(30),
gsm_strength integer,
cdma_dbm integer,
evdo_dbm integer,
rssi integer,
user_lon varchar(30),
user_lat varchar(30),
network_type varchar(30),
Landmark varchar(60),
network_bigtype varchar(30),
province varchar(20),
city varchar(20),
address varchar(100),
type1 integer,
type2 integer
CONSTRAINT signal_strength_pk PRIMARY KEY(id));

```

启动命令行窗口

```
	/usr/crh/current/phoenix-client/bin/sqlline.py hadoop2,hadoop3,hadoop4:/hbase-unsecure
```
命令行中创建数据库

```
CREATE TABLE taxi_info (  TAXI_NUMBER varchar, TAXI_EVENT varchar, TAXI_STATUS varchar, TAXI_TIME varchar, TAXI_LNG varchar, TAXI_LAT varchar, TAXI_SPEED varchar, TAXI_ANGLE varchar, TAXI_GPSSTATUS varcharconstraint pk primary key("TAXI_NUMBER", "TAXI_TIME")) 
psql.py  master,slave1,slave2  /opt/file/phoenix/phoenix/tb_nwQuality.sql
```
MR加载数据

```
执行hdfs目录
export HADOOP_CLASSPATH=$(hbase classpath):$HADOOP_CLASSPATH
hadoop jar /usr/crh/4.9.2.5-1051/phoenix/phoenix-4.6.0-HBase-1.1-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool -t taxi_info -i hdfs://hadoop1:8020/user/traffic/20121018080757.txt -z hadoop2,hadoop3,hadoop4

注意：1.表名区分大小写 2.后缀必须是csv 3.执行本地目录
/usr/crh/current/phoenix-server/bin/psql.py -t TAXI_INFO  -d $',' hadoop2:2181:/hbase /opt/20121018080757.csv
```

```
cd /usr/hdp/current/phoenix-client/bin/ 
./psql.py localhost:2181:/hbase-unsecure /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.sql /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.csv /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT_QUERIES.sql

$PHOENIX_HOME/bin/psql.py localhost:2181 $IMPORT_DDL/all_edges.sql && \ 
$PHOENIX_HOME/bin/psql.py localhost:2181 $IMPORT_DATA/all_edges.csv

/usr/hdp/current/phoenix-client/bin/sqlline.py localhost:2181:/hbase-unsecure CreateTables.sql
/usr/hdp/current/phoenix-client/bin/psql.py -t MOVIES  -d $'\t' localhost:2181:/hbase-unsecure movies.csv
/usr/hdp/current/phoenix-client/bin/psql.py -t RATINGS -d $'\t' localhost:2181:/hbase-unsecure ratings.csv
```

## HBase eclipse开发测试

## HBase 服务器端运行
