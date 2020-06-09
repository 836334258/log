## 事务的ACID属性

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

6. 