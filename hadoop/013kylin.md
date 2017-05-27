# kylin 
## 路径
http://10.0.5.40:7070/kylin
admin/KYLIN
## 数据准备
	hdfs dfs -ls /ordernew
	Found 1 items
	-rw-r--r--   3 hdfs hdfs    1933695 2017-02-21 18:12 /ordernew/order.csv


	CREATE EXTERNAL TABLE IF NOT EXISTS order_data_new (
	orderid string,
	order_date string,
	send_date string,
	send_type string,
	client_id string,
	client_name string,
	client_type string,
	client_region_city string,
	client_region_province string,
	client_country string,
	client_region string,
	product_id string,
	prodcut_category string,
	prodcut_category2 string,
	product_name string,
	money double,
	amount int,
	discount double,
	profit double)	comment 'order data'	row format delimited fields terminated by ','location '/ordernew';
	CREATE EXTERNAL TABLE IF NOT EXISTS order_return (orderid string,status string)	comment 'order return data'	row format delimited fields terminated by ','	location '/order/return/';
	CREATE EXTERNAL TABLE IF NOT EXISTS order_seller (region string,seller_name string)     comment 'order seller data'     row format delimited fields terminated by ','     location '/order/seller/';