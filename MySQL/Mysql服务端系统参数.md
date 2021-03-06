﻿#MySQL系统参数
```
innodb_read_io_threads              设置InnoDB的读写线程数
innodb_write_io_threads             
innodb_purge_threads                用于刷新脏页，减轻master线程压力,innodb1.2开始

innodb_thread_concurrency=64        innodb并发查询的线程数，不是并发连接，也不包括在锁等待的线程，建议设置为64~128，默认为0，表示不限制
innodb_flush_neighbors=0/1			默认1，MySQL8.0默认0。InnoDB在刷新脏页时，是否将相邻的脏页一起刷新。一般如果是机械硬盘，一起刷新可降低随机IO，带来性能提升；但如果是SSD固态硬盘，则可以设置为0，不刷新相邻的脏页，缩短刷新脏页的时间来提升性能。
innodb_io_capacity=200				建议设置为磁盘的IOPS，可以用fio工具查看IOPS。告诉InnoDB的磁盘能力，提升性能。
innodb_max_dirty_pages_pct=75		脏页上限比例，当内存中脏页达到该比例时，强制刷新脏页到磁盘。默认75%
innodb_purge_batch_size=300         每次回收20个undolog页，可动态设置，innodb1.2之前默认20
innodb_max_purge_lag=               控制history list的长度，若长度大于该参数时，其会“延缓”DML的操作
innodb_max_purge_lag_delay=         其用来控制delay的最大毫秒数。也就是当上述计算得到的delay值大于该参数时，将delay设置为innodb_max_purge_lag_delay，避免由于purge操作缓慢导致其他SQL线程出现无限制的等待。
binlog_max_flush_queue_time=        用来控制事务提交时，flush阶段等待的最大时间，用于group commit(一次flush提交多个事务），提高性能，默认0。
innodb_autoinc_lock_mode=0/1/2		主键自增默认，默认1，自增可以采用表锁或者或者基于内存的互斥量自增。0表示表锁，效率低，2表示内存互斥量，效率高，1:在bulk insert采用表锁，simple insert采用内存互斥量
innodb_file_per_table=ON/OFF		默认为ON，一个表存一个单独的文件。为OFF时表示表的数据存在系统共享表空间，这样会导致在drop table 时，不会回收对应的储存空间。推荐为ON。
innodb_log_file_size=536870912		单个redolog的大小，默认48M，一般修改为4个1G大小的文件。redolog是重复使用的，如果空间用完，MySQL会暂停接收请求，开始刷新redolog到数据文件。
innodb_log_files_in_group=2			默认为2，redolog个数，建议修改为4。
innodb_change_buffer_max_size=50    innodb buffer size中，为update等作为写缓冲区的大小，默认为buffer pool的50%。如果是写多读少的场景，可以使用该配置加速写性能。
innodb_online_alter_log_max_size    online ddl时写操作的缓存大小，如果期间写操作较多，超出限制，则抛出异常

innodb_undo_directory               设置undo log文件坐在的路径，默认为.，表示当前innodb储存引擎的目录
innodb_undo_logs                    设置rollback segment的个数，默认128.
innodb_rollback_segment             设置rollback segment的个数，默认128.innodb1.2版本后使用
innodb_undo_tablespaces             设置用来构成rollback segment文件的数量

## 用于统计执行计划中Cardinality的值时，null值的计算方式
innodb_stats_method=nulls_equal/nulls_unequal/nulls_ignored，表示统计是如何对待null值，默认相等，也可以设置不等，忽略等情况

innodb_force_recovery=0             0-6数字选项，默认0，代表启动时，进行所有恢复，如不能恢复，则写错误日志，且可能会宕机。
innodb_flush_nerghbors=0            可禁用flush相邻的脏页，如是机械硬盘，可开启提升性能

innodb_log_buffer_size=16777216     redo log buffer空间，默认16M。会每秒定时flush到磁盘，所以只需缓冲一秒内事务产生的redolog即可
innodb_flush_log_at_trx_commit=0/1/2	每次事务提交时刷新redolog到磁盘。建议设置为1。为0时不刷盘，只写内存；为1时每次commit都直接flush到磁盘；为2时每次提交只write到操作系统的page cache，由后台线程每隔1秒定时flush
如果redo log buffer的大小占到 innodb_log_buffer_size的一半时，后台线程会主动将redo log write到OS的page cache
innodb_flush_method=fdatasync/O_DIRECT  默认fdatasync，指innodb调用write时只写入os cache，由操作系统自行flush到磁盘。O_DIRECT则直接写入磁盘，这样会导致写入操作响应时间变长，但io,cpu消耗变低。

innodb_buffer_pool_instances=1      buffer pool个数，设置多个可以减少内存资源锁定的冲突，提升性能，默认1
innodb_old_blocks_pct=37            Innodb缓冲区采用LRU算法淘汰使用很少的内存页(16k)，默认最新访问的页并不是放在LRU的头部，而是在LRU列表尾部37%的位置
innodb_old_blocks_time=1000         表示新数据页在加入到LRU列表mid位置后多久才能加入LRU中的热点部位

innodb_lock_wait_timeout            innodb默认的锁超时时间，默认50s
innodb_rollback_on_timeout=off      在锁超时后，默认不回滚事务.事务在锁超时后，既不会自动rollback，也不会自动commit，需要手动操作。
innodb_deadlock_detect=ON/OFF       死锁检测

innodb_sort_buffer_size             mysql会为每个线程分配一个排序缓冲区，如果要排序的数据量小于该值，则在内存中排序(性能好)，否则将借助磁盘临时文件排序
innodb_adaptive_hash_index=ON       自适应哈希索引，默认开启
innodb_adaptive_hash_index_parts=8  hash索引的分区，默认8，最大512，加大可适当提高性能，降低索引的锁竞争。
query_cache_type=ON/OFF/DEMAND      开启/关闭查询缓存，一般建议关闭。查询缓存是表级别的，一个update会清空整个表的查询缓存，如果update频繁，则导致查询缓存弊大于利。可使用DEMAND配合'select SQL_CACHE * from table'显示指定。mysql8.0以后已经关闭查询缓存的功能

max_connections					    默认256，服务端接收的最大连接数，超过该限制，报 Too many connections。
wait_timeout						默认28800，8小时。指连接在空闲8小时后将断开。
binlog_cache_size=32768			    单个线程的binlog缓存32M，单个事务是一次性写入binlog。如果单个事务过大，超过该限制，则会使用磁盘空间暂存。
sync_binlog=0/1/N					刷新binlog到磁盘文件。为0时表示只调用write写入操作系统缓存，不刷新fsync到磁盘；为1时每次事务刷新到磁盘；为N时表示每N个事务fsync刷新到磁盘。建议设置为1
binlog_group_commit_sync_delay=0	组提交延迟微秒数。减少binlog写盘次数，提升IO性能。为0时binlog_group_commit_sync_no_delay_count也不生效。
binlog_group_commit_sync_no_delay_count=0	组提交延迟的个数，与binlog_group_commit_sync_delay有一个满足则提交。

log_slave_updates=ON                设置备库在执行relaylog后也生成binlog，适用于双主集群，双主集群在备份复制时，使用binlog中的server id判断去重
long_query_time=10                  默认10秒
log_slow_queries=ON                 开启慢查询
log_queries_not_using_indexes=ON    开启记录没有使用索引的查询到慢查询日志
log_throttle_not_using_indexes=0    表示每分钟允许记录到slow log的且未使用索引的SQL语句次数。该值默认为0，表示没有限制
long_query_io                       将超过指定逻辑IO次数的SQL语句记录到slow log中。该值默认为100
slow_query_type                     表示启用slow log的方式，可选值为0-3，0表示不将SQL语句记录到slow log,3表示根据运行时间及逻辑IO次数将SQL语句记录到slow log

join_buffer_size=262144             连表缓冲区大小，默认256k
read_rnd_buffer_length              回表查询时，先根据索引查出对应的主键id(一般索引对应的主键索引可能是无序的，造成磁盘随机读，性能低)，放在read_rnd_buffer中，然后排序id，在到主键上顺序查询
tmp_table_size=16777216             内存临时表大小，默认16M
secure_file_priv=/path              


####################
数据库遇到类似 CPU 问题，可以完成以下操作，对追溯问题根源是很有帮助的：
vmstat 1 1000
top -Hu mysql
perf top -a -g
show engine innodb status \G
show processlist
重启前打pstack 日志（确定重启前才能打，其他时候不能乱打
#####################

WAL: MySQL性能好的关键是Write Ahead Log(redo log),将随机写磁盘变为顺序写；  
change buffer: 普通索引才可用，每次索引的修改直接在change buffer中修改，不用去磁盘读；  
group commit: redo log 和 binlog 都可以组提交，合并多个事务的log一起写磁盘，减少flush磁盘的次数
事务的redo log和binlog的流程：redo log prepare(write) -- binlog(write) -- redo log prepare(flush) -- binlog(flush) -- redo log commit(write)
这样将flush的时机延后，可以稍微等待，将多个事务的log一起flush到磁盘
```
