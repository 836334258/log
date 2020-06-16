

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

5. `show profiles`显示当前执行的语句和时间

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

