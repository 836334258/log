

- 原子性
  - 对其数据的修改，要么全执行，要么全不执行
- 一致性
  - 事务开始和完成时，数据必须保持一致
- 隔离性
  - 事务处理过程中的中间状态对外不可见
- 持久性
  - 事务完成后，对数据的修改是永久的

## 索引优化

1. 最佳左前缀法则
2. 不要在索引列上做任何操作，比如函数操作
3. 在`where`中使用了范围，则范围右边的列失效
4. 尽量使用覆盖索引(索引列和查询列顺序一致)，少用select *
5. 在使用不等于`!=`或者`<>`时，无法使用索引，使用的是全表扫描
   - `!=`和`<>`推荐使用`<>`，因为ANSI标准中用的是`<>` 
6. `is null` 和`is not null`无法使用索引
7. `like` 以通配符开头的`%xyz`，索引会失效，变成全表扫描
8. 隐式类型转换会导致索引失效
9. 少用`or`

## 常用操作

- 更改密码 `mysqladmin -uroot -p原密码 password 123456`

- 配置远程登录

  ``````
  mysql> create user 'root'@'%' identified by '你自己的mysql密码';
  mysql> grant all privileges on *.* to 'root'@'%';
  mysql> flush privileges;
  ``````

## 慢查询

1. 查看是否开启慢查询 `show variables like 'slow_query%'`

2. 开启慢查询 my.cnf

   ```mysql
   [mysqld]
   ;key_buffer_size=16M
   slow_query_log=1 
   long_query_time=2
   ```

3. 查询慢查询次数`show STATUS like "slow_queries"`

4. Mysqldumpslow 分析慢查询日志

   ```bash
   root@mysql:/var/log/mysql# mysqldumpslow -s t /var/log/mysql/mariadb-slow.log
   
   Reading mysql slow query log from /var/log/mysql/mariadb-slow.log
   Count: 1  Time=2.00s (2s)  Lock=0.00s (0s)  Rows_sent=1.0 (1), Rows_examined=0.0 (0), Rows_affected=0.0 (0), root[root]@[172.16.238.1]
     SELECT sleep(N)
   
   Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows_sent=0.0 (0), Rows_examined=0.0 (0), Rows_affected=0.0 (0), 0users@0hosts
     mysqld, Version: N.N.N-MariaDB-N:N.N.N+maria~bionic (mariadb.org binary distribution). started with:
     mysqld, Version: N.N.N-MariaDB-N:N.N.N+maria~bionic (mariadb.org binary distribution). started with:
     mysqld, Version: N.N.N-MariaDB-N:N.N.N+maria~bionic-log (mariadb.org binary distribution). started with:
     # Time: N  N:N:N
     # User@Host: root[root] @  [N.N.N.N]
     # Thread_id: N  Schema: test  QC_hit: No
     # Query_time: N.N  Lock_time: N.N  Rows_sent: N  Rows_examined: N
     # Rows_affected: N  Bytes_sent: N
     use test;
     SET timestamp=N;
     SELECT sleep(N);
     mysqld, Version: N.N.N-MariaDB-N:N.N.N+maria~bionic-log (mariadb.org binary distribution). started with:
     # Time: N  N:N:N
     # User@Host: root[root] @  [N.N.N.N]
     # Thread_id: N  Schema: test  QC_hit: No
     # Query_time: N.N  Lock_time: N.N  Rows_sent: N  Rows_examined: N
     # Rows_affected: N  Bytes_sent: N
     use test;
     SET timestamp=N;
     SELECT sleep(N)
   
   ```

5. `show profiles`显示15条语句之前的执行的语句和时间

## 水平拆分和垂直拆分

垂直拆分是吧不同的表拆分到不同的数据库中，水平拆分是吧同一个表拆到不同的数据库中

## 数据备份

- 类型
  - 完全备份
  - 部分备份
    - 增量备份 （备份自上一次备份以来（增量或完全）以来备份的数据；特点：节约空间，还原麻烦）
    - 差异备份（备份自上一次完全备份以来变化的数据。特点：让费空间，还原比增量备份简单）
- 方式
  - 热备份（备份时，数据库的读写操作都不受影响）
  - 温备份（备份时，数据库读操作可以执行，但是不能执行写操作）
  - 冷备份（备份时，数据库不能执行读写操作）

## 索引

- 聚集索引（主键索引）就是按照每张表的主键构造一颗B+树，同时叶子节点存放的即为整张表的记录数据。聚集索引的叶子节点成为数据页。
- 覆盖索引：SQL只需要通过索引就可以返回查询所需要的数据，而不必通过二级索引查到主键之后再去查询数据

## 注意点

- mysql的 varchar是以字符为单位的

- 保存ip地址时可以使用int类型保存，mysql函数 INET_ATON('127.0.0.1')，INET_NTOA('2130706433')

- `SELECT * FROM tests ORDER BY CONVERT(title USING gbk);` 按中文排序

- 尽量不要使用null类型，因为会mysql会增加一层判断

- 查询or时可以

  ```mysql
  select id from t where num=10
  
  union all
  
  select id from t where num=20
  ```

## 函数和语句

##### extract() 

```mysql
SELECT EXTRACT(YEAR_MONTH FROM NOW());
202006
```

##### dayname()

```mysql
SELECT DAYNAME(NOW());
Monday
```

##### if()

```mysql
SELECT IF(RAND()>0.5,TRUE,FALSE)
0
```

##### date_add()&&interval

```mysql
SELECT DATE_ADD(NOW(),INTERVAL 1 MONTH);
2020-07-15 17:56:18
```

##### is_ipv4()

```mysql
SELECT is_ipv4('10.0.5.9');
1
```

##### json_array

```mysql
SELECT JSON_ARRAY('id',1,'name','king');
["id", 1, "name", "king"]
```

##### json_object

```mysql
SELECT JSON_OBJECT('id',1,'name','king');
{"id": 1, "name": "king"}
```

##### json_array_append

```mysql
["id", 1, "name", "king"]
SELECT JSON_ARRAY_APPEND(@j,'$[0]', 'age2');
[["id", "age2"], 1, "name", "king"]
```

##### json_array_insert

```mysql
SELECT JSON_ARRAY_INSERT(@j, '$[1]', "ccc");
["id", "ccc", 1, "name", "king"]
```

##### json_contains 查找对应字符

```mysql
set @j='{"id": 1, "name": "king"}';
SELECT JSON_CONTAINS(@j, '1','$.id'); -- 必须是字符串1
1
```

##### json_contains_path 判断对应项存不存在

```mysql
set @j='{"id": 1, "name": "king"}';
SELECT JSON_CONTAINS_PATH(@j, 'one', '$.id');
1
SELECT JSON_CONTAINS_PATH(@j, 'all', '$.id','$.name1');
0
```

##### Json_depth 查找json最深深度

```mysql
SELECT JSON_DEPTH(@j);
2
```

##### json_extract 指定索引查找

```mysql
SELECT JSON_EXTRACT(@j, '$.id','$.name');
[1, "king"]
```

##### json_keys

```mysql
SELECT JSON_KEYS(@j);
["id", "name"]
```

##### json_length

```mysql
SELECT JSON_LENGTH(@j);
2
```

##### json_object

```mysql
SELECT JSON_OBJECT('a',1,'b',2);
{"a": 1, "b": 2}
```

##### JOSN_OBERLAPS(INTRODUCED 8.0.17) 只要一个键值对匹配，返回1

```mysql
set @j=JSON_OBJECT("a",1,"b",2);
set @k=JSON_OBJECT("a",1,"c",2);

SELECT JSON_OVERLAPS(@j,@k);
1
```

##### json_pretty

```mysql
mysql root@localhost:(none)> select json_pretty(@j)
+------------------+
| json_pretty(@j)  |
+------------------+
| {                |
|   "id": 1,       |
|   "name": "king" |
| }                |
+------------------+
```

##### json_remove

```mysql
SELECT JSON_REMOVE(@j,'$.a');
{"b": 2}
```

##### json_replace 只能替换@j里面已有的值

```mysql
SELECT JSON_REPLACE(@j, '$.a', 10,'$.b',20)
{"a": 10, "b": 20}
```

##### json_set 可以随意设置值

```mysql
SELECT JSON_SET(@j, '$.a', 10,'$.b',20,'$.c',30);
{"a": 10, "b": 20, "c": 30}
```

##### json_valid 检验json格式

```mysql
SELECT JSON_VALID('{"name"1}');
0
```

##### last_day()返回这个月的最后一天

```mysql
SELECT LAST_DAY(NOW());
2020-06-30
```

##### last_insert_id()

```mysql
SELECT LAST_INSERT_ID();
2
```

##### left()

```mysql
SELECT LEFT("abcd",2);
ab
```

##### lpad()

```mysql
SELECT LPAD("hi",4,"??");
??hi
```

##### member of  introduced 8.0.17)

```mysql
SELECT 7 member of ('[7,1,3]');
1
```

##### random_bytes

```mysql
SELECT random_bytes(8);
m+õ·»L
```



#### Mysql语句

```mysql
-- union 合并结果集合 查询所有,相同的数据不显示出来，但是两张表的结构必须相同，查询的字段也要相同
SELECT * FROM dx_users WHERE id<10
UNION
SELECT * FROM dx_users WHERE id<11
;
```

```mysql
-- union all 查询所有,相同的数据也会显示出来，但是两张表的结构必须相同，查询的字段也要相同
SELECT * FROM dx_users WHERE id<10
UNION all
SELECT * FROM dx_users WHERE id<11
;
```

​	

```mysql
-- 连接查询，会产生笛卡尔集
SELECT * FROM dx_users,dx_staffs
```

```mysql
-- 选择连接查询
SELECT * FROM dx_users du,dx_staffs ds WHERE du.id=ds.user_id ORDER BY du.id
```

```mysql
-- 内连接查询 内连接与多表联查约束主外键相同，只是写法改变
SELECT * FROM dx_users INNER JOIN dx_staffs WHERE dx_users.id=dx_staffs.user_id ORDER BY dx_users.id
```

