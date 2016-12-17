# bigdata_learning
## 集成知识
###GIT
    $ echo "# bigdata_learning" >> README.md
    $ git init
    $ git add README.md
    $ git commit -m "first commit"
    $ git remote add origin https://github.com/zhang637/bigdata_learning.git
    $ git push -u origin master
###LINUX
    cat /etc/issue
    cat /etc/redhat-release
    lsblk 
    du -h --max-depth=1
##资源准备
    10.0.1.101-10.0.1.105    root/centos68
    http://public-repo-1.hortonworks.com/ARTIFACTS/jdk-7u67-linux-x64.tar.gz
    http://public-repo-1.hortonworks.com/ambari/centos6/a/ambari-1.7.0-centos6.tar.gz 
    http://public-repo-1.hortonworks.com/HDP/centos6/HDP-2.2.0.0-centos6-rpm.tar.gz 
    http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.20/repos/centos6/HDP-UTILS-1.1.0.20-centos6.tar.gz

    nohup wget -c ://public-repo-1.hortonworks.com/ARTIFACTS/jdk-8u60-linux-x64.tar.gz > jdk.log 2>&1 & 
    nohup wget -c http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.2.2.0/ambari-2.2.2.0-centos6.tar.gz > ambari.log 2>&1 &  
    nohup wget -c http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.4.1.0/ ambari-2.4.1.0-centos6.tar.gz > ambari.log 2>&1 &  
    nohup wget -c http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.4.2.0/HDP-2.4.2.0-centos6-rpm.tar.gz > hdp.log 2>&1 & 
    nohup wget -c http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.20/repos/centos6/HDP-UTILS-1.1.0.20-centos6.tar.gz > hdp-util.log 2>&1 &
#虚拟机网络设置
如果没有看到“Bringing up interface eth1: [ OK ]”这样的反馈信息，那有可能是因为 eth1 没有安装好。试着执行下面的指令让 CentOS 自动检测新硬件并安装：
　　start_udev
##系统环境配置
###IP设置
    ![](img/1.jpg)
###主机名HOST配置
    vi /etc/hosts
    vi /etc/sysconfig/network
    hostname xxx
###安全Selinux关闭
     vi /etc/selinux/config
     setenforce 0
###防火墙iptables关闭
    # service iptables stop
        iptables：将链设置为政策 ACCEPT：filter                    [确定]
        iptables：清除防火墙规则：                                 [确定]
        iptables：正在卸载模块：                                   [确定]
    # chkconfig --list ip6tables
        ip6tables      	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
    # chkconfig ip6tables off
    # service ip6tables stop
        ip6tables：将 chains 设置为 ACCEPT 策略：filter            [确定]
        ip6tables：清除防火墙规则：                                [确定]
        ：正在卸载模块：                                           [确定]
###无密码登陆ssh-keygen
    ssh-keygen  直接一路回车
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@hdp2
###时钟同步
    #vim /etc/ntp.conf
    #server 210.72.145.44 perfer         # 中国国家受时中心
    #server 202.112.10.36                # 1.cn.pool.ntp.org
    #server 59.124.196.83                # 0.asia.pool.ntp.org
    ntpdate 210.72.145.44    
###mysql数据库
yum -y install mysql-server
service mysqld start
mysqladmin -uroot password Passw0rd
create user 'root'@'hdp2' identified by 'Passw0rd';
grant all privileges on *.* to 'root'@'hdp2';
flush privileges;
###JDK安装
export PATH=$JAVA_HOME/bin/:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

##Ambari安装部署
### 离线安装源
    yum install httpd
    service httpd start
    mkdir  /var/www/html/ambari
    mkdir  /var/www/html/hdp-util
    mkdir  /var/www/html/hdp
    tar -xzvf ambari-2.2.2.0-centos6.tar.gz -C /var/www/html/ambari
    tar -xzvf HDP-UTILS-1.1.0.20-centos6.tar.gz  -C /var/www/html/ambari
    yum install createrepo
    进入/var/www/html/ambari 目录 -->  createrepo  ./    
    tar -zxvf HDP-2.2.0.0-centos6-rpm.tar.gz -C /var/www/html/hdp
    进入/var/www/html/ambari 目录 -->  createrepo  ./  

    wget -nv http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.5.0.0/hdp.repo  
    vim /etc/yum.repos.d/ambari.repo 新建repo文件
        [Updates-ambari-2.4.1.0]
        name=ambari-2.4.1.0 - Updates
        baseurl=http://hdp2/AMBARI-2.4.1.0/centos6/2.4.1.0-22/
        gpgcheck=0
        enabled=1
        priority=1

    vim /etc/yum.repos.d/hdp.rep
    [HDP-2.5.0.0]
    name=HDP Version - HDP-2.5.0.0
    baseurl=http://hdp2/hdp/HDP/centos6/
    gpgcheck=0
    enabled=1
    priority=1
    [HDP-UTILS-1.1.0.21]
    name=HDP-UTILS Version - HDP-UTILS-1.1.0.21
    baseurl=http://hdp2/hdp/HDP-UTILS-1.1.0.21/repos/centos6/
    gpgcheck=0
    enabled=1
    priority=1
### 安装ambari server
    yum -y install ambari-server
    ambari-server setup
    ambari-server start
### 基于ambari配置hadoop
    进入目录 /var/lib/ambari-server/resources/stacks/HDP/2.2/repos
    vi repoinfo.xml
        <os type="redhat6">
            <repo>
              <baseurl>http://hdp1/hdp/</baseurl>
              <repoid>HDP-2.2</repoid>
              <reponame>HDP</reponame>
            </repo>
            <repo>
              <baseurl>http://hdp1/ambari</baseurl>
              <repoid>HDP-UTILS-1.1.0.20</repoid>
              <reponame>HDP-UTILS</reponame>
            </repo>
        </os>

        yum clean 
        yum makecache
### 修改配置文件
    http://hadoop/hdp/HDP/centos6/2.x/GA/2.2.0.0/
    http://192.168.0.120/ambari/HDP-UTILS-1.1.0.20/repos/centos6/

ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
create database oozie;
create user 'oozie'@'hdp2' identified by 'Passw0rd';
grant all privileges on *.* to 'oozie'@'hdp2';
flush privileges;
create database hive;
create user 'hive'@'hdp2' identified by 'Passw0rd';
grant all privileges on *.* to 'hive'@'hdp2';
flush privileges;

##HDFS
sudo -u hdfs hdfs dfs -ls /

##MapReduce测试
    useradd hdpclient
    usermod -a -G users hdpclient
    sudo su - hdfs
    hdfs dfs -mkdir /user/hdpclient
    hdfs dfs -chown hdpclient:hdpclient /user/hdpclient
    hdfs dfs -chmod -R 755 /user/hdpclient
    su - hdpclient
    hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-*.jar teragen 10000 tmp/teragenout
    hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-*.jar terasort tmp/teragenout tmp/terasortout
##Hbase 测试
su - hbase
hbase shell
status 'detailed'
hbase thrift start -p <port> --infoport <infoport>
hbase rest start -p <port> --infoport <infoport>
hbase-daemon.sh start thrift -p <port> --infoport <infoport>

13/12/11 09:45:33 INFO client.ZooKeeperRegistry: ClusterId read in ZooKeeper is null

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
##pheonix 安装测试
<property>
    <name>hbase.defaults.for.version.skip</name>
    <value>true</value>
</property>
hbase.regionserver.wal.codec : org.apache.hadoop.hbase.regionserver.wal.WALCellCodec  --> org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec
hbase.region.server.rpc.scheduler.factory.class : org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory
hbase.rpc.controllerfactory.class : org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory
phoenix.functions.allowUserDefinedFunctions : true

cd /usr/hdp/current/phoenix-client/bin/ 
./psql.py localhost:2181:/hbase-unsecure /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.sql /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT.csv /usr/hdp/current/phoenix-client/doc/examples/WEB_STAT_QUERIES.sql

