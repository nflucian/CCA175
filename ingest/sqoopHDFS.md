## Basic Sqoop Commands

List databases
```
$ sqoop list-databases \
--connect jdbc:mysql://quickstart.cloudera:3306 \
--username retail_dba --password cloudera
```

List _retail_db_ tables
```
$ sqoop list-tables \
--connect jdbc:mysql://localhost:3306/retail_db \
--username retail_dba --password cloudera
```

Query Evaluation
```
$ sqoop eval \
--connect jdbc:mysql://localhost:3306/retail_db \
--username retail_dba --password cloudera \
--query "SELECT COUNT(1) FROM products"
```
```
$ sqoop eval \
--connect jdbc:mysql://localhost:3306/retail_db \
--username retail_dba --password cloudera \
--query "SELECT * FROM orders LIMIT 5"
```

## Import data

Import all tables
```
$ sqoop import-all-tables \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera
```
```
$ hdfs dfs -ls /user/cloudera/
Found 6 items
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:53 categories
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:54 customers
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:54 departments
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:55 order_items
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:55 orders
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 09:56 products
```
Import all tables as avro files in a warehouse directory
```
$ sqoop import-all-tables \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera \
--warehouse-dir /user/cloudera/warehouse/ \
--as-avrodatafile
```
```
$ hdfs dfs -ls /user/cloudera/warehouse
Found 6 items
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:34 /user/cloudera/warehouse/categories
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:34 /user/cloudera/warehouse/customers
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:35 /user/cloudera/warehouse/departments
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:35 /user/cloudera/warehouse/order_items
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:36 /user/cloudera/warehouse/orders
drwxr-xr-x   - cloudera cloudera          0 2018-03-19 10:36 /user/cloudera/warehouse/products

$ hdfs dfs -ls /user/cloudera/warehouse/categories
Found 5 items
-rw-r--r--   1 cloudera cloudera          0 2018-03-19 10:34 /user/cloudera/warehouse/categories/_SUCCESS
-rw-r--r--   1 cloudera cloudera        776 2018-03-19 10:34 /user/cloudera/warehouse/categories/part-m-00000.avro
-rw-r--r--   1 cloudera cloudera        759 2018-03-19 10:34 /user/cloudera/warehouse/categories/part-m-00001.avro
-rw-r--r--   1 cloudera cloudera        762 2018-03-19 10:34 /user/cloudera/warehouse/categories/part-m-00002.avro
-rw-r--r--   1 cloudera cloudera        725 2018-03-19 10:34 /user/cloudera/warehouse/categories/part-m-00003.avro
```
Import table _as a text file_
```
$ sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera \
--table departments \
--target-dir /user/cloudera/sqoop-import/departments_as_txt \
--as-textfile
```
```
$ hdfs dfs -ls /user/cloudera/sqoop-import/departments_as_txt
Found 5 items
-rw-r--r--   1 cloudera cloudera          0 2018-03-19 10:57 /user/cloudera/sqoop-import/departments_as_txt/_SUCCESS
-rw-r--r--   1 cloudera cloudera         21 2018-03-19 10:57 /user/cloudera/sqoop-import/departments_as_txt/part-m-00000
-rw-r--r--   1 cloudera cloudera         10 2018-03-19 10:57 /user/cloudera/sqoop-import/departments_as_txt/part-m-00001
-rw-r--r--   1 cloudera cloudera          7 2018-03-19 10:57 /user/cloudera/sqoop-import/departments_as_txt/part-m-00002
-rw-r--r--   1 cloudera cloudera         22 2018-03-19 10:57 /user/cloudera/sqoop-import/departments_as_txt/part-m-00003

$ hdfs dfs -cat /user/cloudera/sqoop-import/departments_as_txt/*
2,Fitness
3,Footwear
4,Apparel
5,Golf
6,Outdoors
7,Fan Shop
```
Import orders table and let sqoop to delete target dir if exists and overriding the fields delimiter to $
```
$ sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera \
--table orders \
--target-dir /user/cloudera/orders \
--delete-target-dir \
--fields-terminated-by '$'
```
Conditional Import
```
$ sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera \
--table orders \
--target-dir /user/cloudera/sqoop-import/orders \
--where "order_status='COMPLETE'"
```
Conditional Import and Append to existing file
```
$ sqoop import \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba --password cloudera \
--table orders \
--target-dir /user/cloudera/sqoop-import/orders \
--where "order_status='COMPLETE'" \
--append
```
Import query results
```
$ sqoop import \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --query "SELECT * FROM orders JOIN order_items ON (orders.order_id = order_items.order_item_order_id AND orders.order_status='COMPLETE') where \$CONDITIONS" \
  --target-dir /user/cloudera/sqoop-import/query/order_join \
  --split-by order_id
```
Import when fields are enclosed in (') and separator of the field should be (~) and line needs to be terminated by :
```
$ sqoop import \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table departments \
  --target-dir /user/cloudera/sqoop-import/departments_enclosedby \
  --fields-terminated-by '~' \
  --lines-terminated-by ':' \
  --enclosed-by '"' \
  --escaped-by \\
```
```
$ hdfs dfs -cat /user/cloudera/sqoop-import/departments_enclosedby/*
"2"~"Fitness":"3"~"Footwear":"4"~"Apparel":"5"~"Golf":"6"~"Outdoors":"7"~"Fan Shop":
```

## Export data

Export data from HDFS to mysql table
```
$ mysql -h quickstart.cloudera -u retail_dba -c retail_db -pcloudera

mysql> CREATE TABLE `order_items_export` (
    `order_item_id` int(11) NOT NULL AUTO_INCREMENT,
    `order_item_order_id` int(11) NOT NULL,
    `order_item_product_id` int(11) NOT NULL,
    `order_item_quantity` tinyint(4) NOT NULL,
    `order_item_subtotal` float NOT NULL,
    `order_item_product_price` float NOT NULL,
    PRIMARY KEY (`order_item_id`)
) ENGINE=InnoDB AUTO_INCREMENT=172199 DEFAULT CHARSET=utf8;

mysql> CREATE TABLE `orders_export` (
    `order_id` int(11) NOT NULL AUTO_INCREMENT,
    `order_date` datetime NOT NULL,
    `order_customer_id` int(11) NOT NULL,
    `order_status` varchar(45) NOT NULL,
    PRIMARY KEY (`order_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=68884 DEFAULT CHARSET=utf8;
```
```
// using mysqlimport tool

$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table order_items_export \
  --export-dir /user/cloudera/order_items \
  --direct
```
```
// using standard JDBC

$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table order_items_export \
  --export-dir /user/cloudera/order_items \
  --batch
```
Export data delimited by _$_
```
$ hdfs dfs -cat /user/cloudera/orders/* | head
1$2013-07-25 00:00:00.0$11599$CLOSED
2$2013-07-25 00:00:00.0$256$PENDING_PAYMENT
3$2013-07-25 00:00:00.0$12111$COMPLETE
4$2013-07-25 00:00:00.0$8827$CLOSED
5$2013-07-25 00:00:00.0$11318$COMPLETE
```
```
$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table orders_export \
  --export-dir /user/cloudera/orders \
  --input-fields-terminated-by '$'
```
Export specific columns (the other columns in the mysql table has default constraint)
```
$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table orders_export \
  --export-dir /user/cloudera/orders \
  --input-fields-terminated-by '$'
  --columns order_id,order_status
```
Updating existing data
```
$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table order_items_export \
  --export-dir /user/cloudera/order_items \
  --update-key order_item_id \
  --update-mode updateonly
```
Update & Insert
```
$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table order_items_export \
  --export-dir /user/cloudera/order_items \
  --update-key order_item_id \
  --update-mode allowinsert
```
Export data when fields are enclosed in (') and separator of the field should be (~) and line needs to be terminated by :
```
$ sqoop export \
  --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
  --username retail_dba --password cloudera \
  --table departments_export \
  --export-dir /user/cloudera/sqoop-import/departments_enclosedby \
  --fields-terminated-by '~' \
  --lines-terminated-by ':' \
  --enclosed-by '"' \
  --escaped-by \\
```