# Hive知识总结
## 创建外部表
```
  CREATE EXTERNAL TABLE IF NOT EXISTS traffic (
     cid String, eventtype int,status int,ctime string,langitude double,lantitude  double, cspeed double,direction int,valstatus int)
     comment 'traffic data'
     row format delimited fields terminated by ','
     location '/user/traffic/oneM';
```
```
CREATE EXTERNAL TABLE `traffic`(
  `cid` string, 
  `eventtype` int, 
  `status` int, 
  `ctime` string, 
  `langitude` double, 
  `lantitude` double, 
  `cspeed` double, 
  `direction` int, 
  `valstatus` int)
COMMENT 'traffic data'
ROW FORMAT DELIMITED 
  FIELDS TERMINATED BY ',' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://zylhdp:8020/user/traffic/oneM'
TBLPROPERTIES (
  'COLUMN_STATS_ACCURATE'='false', 
  'numFiles'='0', 
  'numRows'='-1', 
  'rawDataSize'='-1', 
  'totalSize'='0', 
  'transient_lastDdlTime'='1486475671')
  ```
  
  