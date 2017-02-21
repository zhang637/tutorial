# HDFS&MR
## HDFS
	sudo -u hdfs hdfs dfs -ls /
添加linux用户
    
    useradd hdpclient
    usermod -a -G users hdpclient
为用户创建hadoop目录    

    sudo su - hdfs
    hdfs dfs -mkdir /user/hdpclient
    hdfs dfs -chown hdpclient:hdpclient /user/hdpclient
    hdfs dfs -chmod -R 755 /user/hdpclient
	
	sudo -u hdfs hdfs dfs -copyFromLocal /opt/oneM/* /user/traffic/
用户在自己目录下执行命令    

    su - hdpclient
## MapReduce测试
    hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-*.jar teragen 10000 tmp/teragenout
    hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-*.jar terasort tmp/teragenout tmp/terasortout
    sudo -u hdfs hadoop jar /opt/wordcount-1.0-SNAPSHOT.jar wordcount.WordCount /user/wordcount/in /user/wordcount/out


## 
安装 hadoop-hdfs-fuse
	
	yum install -y  hadoop-hdfs-fuse

挂载
	
	mkdir /HDFS
	hadoop-fuse-dfs   dfs://hadoop1:8020 /HDFS -obig_writes



