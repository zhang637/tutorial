#HBASE
##HBase 基础
create 'test1', 'lf', 'sf'
				-- lf: column family of LONG values (binary value)
				-- sf: column family of STRING values
				-- 一个用户（userX），在什么时间（tsX），作为rowkey
				-- 对什么产品（value：skuXXX），做了什么操作作为列名，比如，c1: click from homepage; c2: click from ad; s1: search from homepage; b1: buy

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

import org.apache.hadoop.hbase.filter.CompareFilter
import org.apache.hadoop.hbase.filter.SubstringComparator
import org.apache.hadoop.hbase.filter.RowFilter

scan 'test1', {FILTER => RowFilter.new(CompareFilter::CompareOp.valueOf('EQUAL'), SubstringComparator.new('ts3'))}

import org.apache.hadoop.hbase.filter.RegexStringComparator

put 'test1', 'user2|err', 'sf:s1', 'sku999'
scan 'test1', {FILTER => RowFilter.new(CompareFilter::CompareOp.valueOf('EQUAL'),RegexStringComparator.new('^user\d+\|ts\d+$'))}

import org.apache.hadoop.hbase.filter.CompareFilter
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter
import org.apache.hadoop.hbase.filter.SubstringComparator
import org.apache.hadoop.hbase.util.Bytes
scan 't1', { COLUMNS => 'family:qualifier', FILTER =>
    SingleColumnValueFilter.new
        (Bytes.toBytes('family'),
         Bytes.toBytes('qualifier'),
         CompareFilter::CompareOp.valueOf('EQUAL'),
         SubstringComparator.new('somevalue'))
}

put 'test1', 'user1|ts9', 'sf:b1', 'sku1'
scan 'test1', FILTER=>"ColumnPrefixFilter('b1') AND ValueFilter(=,'binary:sku1')"
scan 'test1', {COLUMNS => 'sf:b1', FILTER => SingleColumnValueFilter.new(Bytes.toBytes('sf'), Bytes.toBytes('b1'), CompareFilter::CompareOp.valueOf('EQUAL'), Bytes.toBytes('sku1'))}

-- binary value --

org.apache.hadoop.hbase.util.Bytes.toString("Hello HBase".to_java_bytes)

org.apache.hadoop.hbase.util.Bytes.toString("\x48\x65\x6c\x6c\x6f\x20\x48\x42\x61\x73\x65".to_java_bytes)

-- 用户userX，作为rowkey，他的各种设备（brwoser, app, pc）作为列名，所对应的cookie_id作为value (长整型变量)
put 'test1', 'user1', 'lf:browser1', "\x00\x00\x00\x00\x00\x00\x00\x02"
put 'test1', 'user1', 'lf:app1', "\x00\x00\x00\x00\x00\x00\x00\x0F"
put 'test1', 'user1', 'lf:app2', "\x00\x00\x00\x00\x00\x00\x00\x10"
put 'test1', 'user2', 'lf:app1', "\x00\x00\x00\x00\x00\x00\x00\x11"
put 'test1', 'user2', 'lf:pc1', "\x00\x00\x00\x00\x00\x00\x00\x12"

scan 'test1', STOPROW=>'user2', FILTER=>"( ColumnPrefixFilter('app') AND ValueFilter(>,'binary:\x00\x00\x00\x00\x00\x00\x00\x0F') )"
scan 'test1', LIMIT => 10, FILTER=>"( ColumnPrefixFilter('app') AND ValueFilter(>,'binary:\x00\x00\x00\x00\x00\x00\x00\x0F') )"

alter 'test1', NAME => 'cf', METHOD => 'delete'
alter 'test1', 'delete' => 'cf'
alter 'test1', NAME => 'cf'

-- 用户userX，作为rowkey，在什么时间（timestamp）作为列名，访问了什么页面的id作为value：page_id (整型变量)
put 'test1', 'user1', 'cf:1399999999', "\x00\x00\x00\x09"
put 'test1', 'user1', 'cf:1400000000', "\x00\x00\x00\x08"
put 'test1', 'user1', 'cf:1400000001', "\x00\x00\x00\x07"
put 'test1', 'user1', 'cf:1400000002', "\x00\x00\x20\xFB"
put 'test1', 'user2', 'cf:1500000000', "\x00\x00\x00\x11"
put 'test1', 'user2', 'cf:1500000001', "\x00\x00\x20\xFC"


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



##HBase eclipse开发测试
##HBase 服务器端运行