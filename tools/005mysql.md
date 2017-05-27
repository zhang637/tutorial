# Mysql
## 数据备份
--opt选项会对转储过程进行优化，生成的备份文件会小一点，后的管道操作会进行数据压缩。mysqldump --opt [databasename] [mytable1,mytable2,…]在数据库后接数据表名，只导出指定的数据表，多个数据表可用逗号分隔。
```
#!/bin/shDB_NAME="test"DB_USER="root"DB_PASS="password"BIN_DIR="/usr/local/mysql/bin"BCK_DIR="/home/mysql/backup"DATE=`date +%F`$BIN_DIR/mysqldump --opt -u$DB_USER -p$DB_PASS $DB_NAME | gzip > $BCK_DIR/db_$DATE.gz```
