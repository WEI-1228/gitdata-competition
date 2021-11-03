# Hive学习

## 建表语句

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
  [(col_name data_type [COMMENT col_comment], ...)] 
  [COMMENT table_comment] 
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
  [CLUSTERED BY (col_name, col_name, ...) 
  [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
  [ROW FORMAT row_format] 
  [STORED AS file_format] 
  [LOCATION hdfs_path]
```



```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,page_url STRING,
                       ip STRING COMMENT 'IP Address of the User')
COMMENT 'This is the page view table'
PARTITIONED BY(dt STRING, country STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\001'
STORED AS SEQUENCEFILE;
LOCATION '/table/t1'
```

## 从本地导入数据

```sql
load data local inpath '/home/liujiawei/hivetestdata/xxx.data' into table t_order;
```

## 从hdfs导入数据(会将原始数据移动)

```sql
load data inpath '/uuu.data' into table t_order;
```

## 建立外部表(直接建立在文件夹基础上，不移动文件)

```sql
create external table t_order_ex(int id,name string,rongliang string,price double)
row FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
location '/hive_ext';
```

## 从查询结果生成临时表

```sql
create table tab_ip_ctas
as
select id new_id, name new_name, ip new_ip,country now_country
from tab_ip_ext
sort by new_id;
```

## 向临时表中追加中间结果数据

```sql
create table tab_ip_like like tab_ip

insert overwrite table tab_ip_like
	select * from tab_ip;
```

## 分区

```sql
create table t_order_pt(id int,name string,rongliang string,price double)
partitioned by (month string)
row format delimited fields terminated by '\t';

load data local inpath '/home/liujiawe/hivetestdata/xxx.data' into table t_order_pt partition(month='2020');

select count(*) from t_order_pt where month='2020'
```

## 将表结果导入hdfs中或本地文件中

1. 本地文件直接导出

```sql
insert overwrite local directory '/data/hive/export/student_info' select * from default.student 
```

2. 修改分隔符和换行符

```sql
insert overwrite local directory '/data/hive/export/student_info' 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n'
select * from default.student 
```

3. 导出数据到文件系统

```sql
insert overwrite directory '/data/hive/export/student_info' select * from default.student
```



# HBase学习

## Hbase-shell

打开shell命令行

```bash
$ cd bin
$ ./hbase shell
```



**建表**

```shell
create 'mygirls' ,{NAME=>'base_info',VERSIONS=>3},{NAME=>'extra_info'}
```

**存数据**

```shell
 put 'mygirls','0001','base_info:name','fengjie'
 put 'mygirls','0001','base_info:age','18'
 put 'mygirls','0001','extra_info:boyfriend','huangxiaoming'
```

**取数据**

```shell
get 'mygirls','0001'

get 'mygirls','0001',{COLUMN=>'base_info:name',VERSIONS=>10}
```
