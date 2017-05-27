# sqoop
## 数据
sqoop import --connect jdbc:mysql://hadoop2:3306/mysql --username zyl --password Passw0rd --table mysql --hive-import --hive-table from_mysql_mysqltab --create-hive-table

sqoop export --connect jdbc:mysql://hadoop2:3306/intel_test --username root --password Passw0rd --table test1 -export-dir /user/hive/warehouse/dataDemo

sqoop export --connect jdbc:mysql://10.1.24.210:3306/intel_test --username root --password rootroot --table test1 -export-dir /user/hive/warehouse/datademo --input-fields-terminated-by '&'
 
## 列出mysql数据库中所有数据库
	sqoop list-databases --connect jdbc:mysql://hadoop2:3306/ --username zyl --password Passw0rd
    
## 关系型数据的表结构复制到hive中
	sqoop create-hive-table --connect jdbc:mysql://hadoop2:3306/test --table mysqltable --username root --password Passw0rd --hive-table test
    
## 从关系数据库导入文件到hive中
	sqoop import --connect jdbc:mysql://hadoop2:3306/test --username root --password mysql-password --table t1 --hive-import -m 1
 
## 将hive中的表数据导入到mysql中
	sqoop export --connect jdbc:mysql://hadoop2:3306/test --username root --password Passw0rd --table uv_info --export-dir /user/hive/warehouse/uv/dt=2011-08-03
 
如果报错

```
 11/08/05 10:51:22 INFO mapred.JobClient: Running job: job_201108051007_0010  
11/08/05 10:51:23 INFO mapred.JobClient:  map 0% reduce 0%  
11/08/05 10:51:36 INFO mapred.JobClient: Task Id : attempt_201108051007_0010_m_000000_0, Status : FAILED 
java.util.NoSuchElementException  
        at java.util.AbstractList$Itr.next(AbstractList.java:350)  
        at uv_info.__loadFromFields(uv_info.java:194)  
        at uv_info.parse(uv_info.java:143)  
        at com.cloudera.sqoop.mapreduce.TextExportMapper.map(TextExportMapper.java:79) 
        at com.cloudera.sqoop.mapreduce.TextExportMapper.map(TextExportMapper.java:38) 
        at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:144)  
        at com.cloudera.sqoop.mapreduce.AutoProgressMapper.run(AutoProgressMapper.java:187) 
         at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:647)  
        at org.apache.hadoop.mapred.MapTask.run(MapTask.java:323)  
        at org.apache.hadoop.mapred.Child$4.run(Child.java:270)  
        at java.security.AccessController.doPrivileged(Native Method)  
        at javax.security.auth.Subject.doAs(Subject.java:396)  
at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1127) 
         at org.apache.hadoop.mapred.Child.main(Child.java:264)  
```
此错误的原因为sqoop解析文件的字段与MySql数据库的表的字段对应不上造成的。因此需要在执行的时候给sqoop增加参数，告诉sqoop文件的分隔符，使它能够正确的解析文件字段。
 
hive默认的字段分隔符为'\001'
	
	sqoop export --connect jdbc:mysql://hadoop2:3306/datacenter --username root --password Passw0rd --table uv_info --export-dir /user/hive/warehouse/uv/dt=2011-08-03 --input-fields-terminated-by '\t'
	sqoop import --connect jdbc:oracle:thin:@//db.example.com/foo --table bar


在执行前先下载oracle驱动，将启动放在sqoop/lib下
	
	sqoop list-databases --connect jdbc:oracle:thin:@10.1.24.211:1521:cupoc --username TEST --password oracle
	sqoop import --connect 'jdbc:sqlserver://192.168.1.114; username=sa; password=root;  database=handu' --table test  --hive-import -m 1











