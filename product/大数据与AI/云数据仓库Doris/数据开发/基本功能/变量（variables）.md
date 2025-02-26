本文档主要介绍当前支持的变量（variables）。Doris 中的变量参考 MySQL 中的变量设置。但部分变量仅用于兼容一些 MySQL 客户端协议，并不产生其在 MySQL 数据库中的实际意义。

## 变量设置与查看
### 查看
可以通过 `SHOW VARIABLES [LIKE 'xxx'];` 查看所有或指定的变量。如：
```
SHOW VARIABLES;
SHOW VARIABLES LIKE '%time_zone%';
```

### 设置
部分变量可以设置全局生效或仅当前会话生效。设置全局生效后，后续新的会话连接中会沿用设置值。而设置仅当前会话生效，则变量仅对当前会话产生作用。
仅当前会话生效，通过 `SET var_name=xxx;` 语句来设置。如：

```
SET exec_mem_limit = 137438953472;
SET forward_to_master = true;
SET time_zone = "Asia/Shanghai";
```
全局生效，通过 `SET GLOBAL var_name=xxx;` 设置。如：
```
SET GLOBAL exec_mem_limit = 137438953472
```
>? 
>- 只有 ADMIN 用户可以设置变量的全局生效。
>- 全局生效的变量不影响当前会话的变量值，仅影响新的会话中的变量。

既支持当前会话生效又支持全局生效的变量包括：
- `time_zone`
- `wait_timeout`
- `sql_mode`
- `enable_profile`
- `query_timeout`
- `exec_mem_limit`
- `batch_size`
- `allow_partition_column_nullable`
- `insert_visible_timeout_ms`
- `enable_fold_constant_by_be`

只支持全局生效的变量包括：
- `default_rowset_type`

同时，变量设置也支持常量表达式。如：
```
SET exec_mem_limit = 10 * 1024 * 1024 * 1024;
SET forward_to_master = concat('tr', 'u', 'e');
```

### 在查询语句中设置变量
在一些场景中，我们可能需要对某些查询有针对性的设置变量。
通过使用SET_VAR提示可以在查询中设置会话变量（在单个语句内生效）。如：
```
SELECT /*+ SET_VAR(exec_mem_limit = 8589934592) */ name FROM people ORDER BY name;
SELECT /*+ SET_VAR(query_timeout = 1, enable_partition_cache=true) */ sleep(3);
```

>! 注释必须以/*+ 开头，并且只能跟随在 SELECT 之后。

## 支持的变量
-  `SQL_AUTO_IS_NULL`
用于兼容 JDBC 连接池 C3P0。 无实际作用。
    
- `auto_increment_increment`
用于兼容 MySQL 客户端。无实际作用。
    
- `autocommit`
用于兼容 MySQL 客户端。无实际作用。
    
- `batch_size`
用于指定在查询执行过程中，各个节点传输的单个数据包的行数。默认一个数据包的行数为 1024 行，即源端节点每产生 1024 行数据后，打包发给目的节点。
较大的行数，会在扫描大数据量场景下提升查询的吞吐，但可能会在小查询场景下增加查询延迟。同时，也会增加查询的内存开销。建议设置范围1024至4096。
    
- `character_set_client`
用于兼容 MySQL 客户端。无实际作用。

- `character_set_connection`
用于兼容 MySQL 客户端。无实际作用。

- `character_set_results`
用于兼容 MySQL 客户端。无实际作用。

- `character_set_server`
用于兼容 MySQL 客户端。无实际作用。
    
- `codegen_level`
用于设置 LLVM codegen 的等级。（当前未生效）。
    
- `collation_connection`
用于兼容 MySQL 客户端。无实际作用。

- `collation_database`
用于兼容 MySQL 客户端。无实际作用。

- `collation_server`
用于兼容 MySQL 客户端。无实际作用。

- `delete_without_partition`
设置为 true 时。当使用 delete 命令删除分区表数据时，可以不指定分区。delete 操作将会自动应用到所有分区。
但注意，自动应用到所有分区可能到导致 delete 命令耗时触发大量子任务导致耗时较长。如无必要，不建议开启。
    
- `disable_colocate_join`
控制是否启用  Colocation Join 功能。默认为 false，表示启用该功能。true 表示禁用该功能。当该功能被禁用后，查询规划将不会尝试执行 Colocation Join。

- `enable_bucket_shuffle_join`
控制是否启用 Bucket Shuffle Join 功能。默认为 true，表示启用该功能。false 表示禁用该功能。当该功能被禁用后，查询规划将不会尝试执行 Bucket Shuffle Join。
    
- `disable_streaming_preaggregations`
控制是否开启流式预聚合。默认为 false，即开启。当前不可设置，且默认开启。
    
- `enable_insert_strict`
用于设置通过 INSERT 语句进行数据导入时，是否开启 `strict` 模式。默认为 false，即不开启 `strict` 模式。

- `enable_spilling`
用于设置是否开启大数据量落盘排序。默认为 false，即关闭该功能。当用户未指定 ORDER BY 子句的 LIMIT 条件，同时设置 `enable_spilling` 为 true 时，才会开启落盘排序。该功能启用后，会使用 BE 数据目录下 `doris-scratch/` 目录存放临时的落盘数据，并在查询结束后，清空临时数据。
该功能主要用于使用有限的内存进行大数据量的排序操作。
>! 该功能为实验性质，不保证稳定性，请谨慎开启。
    
- `exec_mem_limit`
用于设置单个查询的内存限制。默认为 2GB，单位为B/K/KB/M/MB/G/GB/T/TB/P/PB, 默认为B。
该参数用于限制一个查询计划中，单个查询计划的实例所能使用的内存。一个查询计划可能有多个实例，一个 BE 节点可能执行一个或多个实例。所以该参数并不能准确限制一个查询在整个集群的内存使用，也不能准确限制一个查询在单一 BE 节点上的内存使用。具体需要根据生成的查询计划判断。
通常只有在一些阻塞节点（如排序节点、聚合节点、Join 节点）上才会消耗较多的内存，而其他节点（如扫描节点）中，数据为流式通过，并不会占用较多的内存。
当出现 `Memory Exceed Limit` 错误时，可以尝试指数级增加该参数，如 4G、8G、16G 等。
    
- `forward_to_master`
用户设置是否将一些show 类命令转发到 Master FE 节点执行。默认为 `true`，即转发。Doris 中存在多个 FE 节点，其中一个为 Master 节点。通常用户可以连接任意 FE 节点进行全功能操作。但部分信息查看指令，只有从 Master FE 节点才能获取详细信息。
如 `SHOW BACKENDS;` 命令，如果不转发到 Master FE 节点，则仅能看到节点是否存活等一些基本信息，而转发到 Master FE 则可以获取包括节点启动时间、最后一次心跳时间等更详细的信息。
当前受该参数影响的命令如下：
	1. `SHOW FRONTENDS;`
转发到 Master 可以查看最后一次心跳信息。
	2. `SHOW BACKENDS;`
转发到 Master 可以查看启动时间、最后一次心跳信息、磁盘容量信息。
	3. `SHOW BROKER;`
转发到 Master 可以查看启动时间、最后一次心跳信息。
	4. `SHOW TABLET;`/`ADMIN SHOW REPLICA DISTRIBUTION;`/`ADMIN SHOW REPLICA STATUS;`
转发到 Master 可以查看 Master FE 元数据中存储的 tablet 信息。正常情况下，不同 FE 元数据中 tablet 信息应该是一致的。当出现问题时，可以通过这个方法比较当前 FE 和 Master FE 元数据的差异。
	5. `SHOW PROC;`
转发到 Master 可以查看 Master FE 元数据中存储的相关 PROC 的信息。主要用于元数据比对。
        
- `init_connect`
用于兼容 MySQL 客户端。无实际作用。
    
- `interactive_timeout`
用于兼容 MySQL 客户端。无实际作用。
    
- `enable_profile`
用于设置是否需要查看查询的 profile。默认为 false，即不需要 profile。
默认情况下，只有在查询发生错误时，BE 才会发送 profile 给 FE，用于查看错误。正常结束的查询不会发送 profile。发送 profile 会产生一定的网络开销，对高并发查询场景不利。
当用户希望对一个查询的 profile 进行分析时，可以将这个变量设为 true 后，发送查询。查询结束后，可以通过在当前连接的 FE 的 web 页面查看到 profile：`fe_host:fe_http_port/query`，
其中会显示最近100条，开启 `enable_profile` 的查询的 profile。
    
- `language`
用于兼容 MySQL 客户端。无实际作用。
    
- `license`
显示 Doris 的 License。无其他作用。

- `load_mem_limit`
用于指定导入操作的内存限制。默认为 0，即表示不使用该变量，而采用 `exec_mem_limit` 作为导入操作的内存限制。
这个变量仅用于 INSERT 操作。因为 INSERT 操作设计查询和导入两个部分，如果用户不设置此变量，则查询和导入操作各自的内存限制均为 `exec_mem_limit`。否则，INSERT 的查询部分内存限制为 `exec_mem_limit`，而导入部分限制为 `load_mem_limit`。
其他导入方式，如 BROKER LOAD，STREAM LOAD 的内存限制依然使用 `exec_mem_limit`。
    
- `lower_case_table_names`
用于控制用户表表名大小写是否敏感。
	- 值为 0 时，表名大小写敏感。默认为0。
	- 值为 1 时，表名大小写不敏感，doris 在存储和查询时会将表名转换为小写。  
优点：在一条语句中可以使用表名的任意大小写形式，下面的sql是正确的：  
```
    mysql> show tables;  
    +------------------+
    | Tables_in_testdb |
    +------------------+
    | cost             |
    +------------------+

    mysql> select * from COST where COst.id < 100 order by cost.id;
```
缺点：建表后无法获得建表语句中指定的表名，`show tables` 查看的表名为指定表名的小写。
	- 值为 2 时，表名大小写不敏感，doris存储建表语句中指定的表名，查询时转换为小写进行比较。
优点：`show tables` 查看的表名为建表语句中指定的表名；  
缺点：同一语句中只能使用表名的一种大小写形式，例如对`cost` 表使用表名 `COST` 进行查询：
 ```
    mysql> select * from COST where COST.id < 100 order by COST.id;
```

该变量兼容 MySQL。需在集群初始化时通过 fe.conf 指定 `lower_case_table_names=`进行配置，集群初始化完成后无法通过`set` 语句修改该变量，也无法通过重启、升级集群修改该变量。
information_schema 中的系统视图表名不区分大小写，当`lower_case_table_names`值为 0 时，表现为 2。

- `max_allowed_packet`
用于兼容 JDBC 连接池 C3P0。 无实际作用。
    
- `max_pushdown_conditions_per_column`
该变量默认置为 -1，表示使用 `be.conf` 中的配置值。如果设置大于 0，则当前会话中的查询会使用该变量值，而忽略 `be.conf` 中的配置值。

- `max_scan_key_num`
该变量默认置为 -1，表示使用 `be.conf` 中的配置值。如果设置大于 0，则当前会话中的查询会使用该变量值，而忽略 `be.conf` 中的配置值。
    
- `net_buffer_length`
用于兼容 MySQL 客户端。无实际作用。

- `net_read_timeout`
用于兼容 MySQL 客户端。无实际作用。
    
- `net_write_timeout`
用于兼容 MySQL 客户端。无实际作用。
    
- `parallel_exchange_instance_num`
用于设置执行计划中，一个上层节点接收下层节点数据所使用的 exchange node 数量。默认为 -1，即表示 exchange node 数量等于下层节点执行实例的个数（默认行为）。当设置大于0，并且小于下层节点执行实例的个数，则 exchange node 数量等于设置值。
在一个分布式的查询执行计划中，上层节点通常有一个或多个 exchange node 用于接收来自下层节点在不同 BE 上的执行实例的数据。通常 exchange node 数量等于下层节点执行实例数量。
在一些聚合查询场景下，如果底层需要扫描的数据量较大，但聚合之后的数据量很小，则可以尝试修改此变量为一个较小的值，可以降低此类查询的资源开销。如在 DUPLICATE KEY 明细模型上进行聚合查询的场景。
    
- `parallel_fragment_exec_instance_num`
针对扫描节点，设置其在每个 BE 节点上，执行实例的个数。默认为 1。
一个查询计划通常会产生一组 scan range，即需要扫描的数据范围。这些数据分布在多个 BE 节点上。一个 BE 节点会有一个或多个 scan range。默认情况下，每个 BE 节点的一组 scan range 只由一个执行实例处理。当机器资源比较充裕时，可以将增加该变量，让更多的执行实例同时处理一组 scan range，从而提升查询效率。
而 scan 实例的数量决定了上层其他执行节点，如聚合节点，join 节点的数量。因此相当于增加了整个查询计划执行的并发度。修改该参数会对大查询效率提升有帮助，但较大数值会消耗更多的机器资源，如 -CPU、内存、磁盘 IO。
    
- `query_cache_size`
用于兼容 MySQL 客户端。无实际作用。
    
- `query_cache_type`
用于兼容 JDBC 连接池 C3P0。 无实际作用。
    
-`query_timeout`
用于设置查询超时。该变量会作用于当前连接中所有的查询语句，以及 INSERT 语句。默认为 5 分钟，单位为秒。

- `resource_group`
暂不使用。

- `send_batch_parallelism`                                                                                                                                                   
用于设置执行 InsertStmt 操作时发送批处理数据的默认并行度，如果并行度的值超过 BE 配置中的 `max_send_batch_parallelism_per_job`，那么作为协调点的 BE 将使用 `max_send_batch_parallelism_per_job` 的值。 
                                                                                                 
- `sql_mode`
用于指定 SQL 模式，以适应某些 SQL 方言。
    
- `sql_safe_updates`
用于兼容 MySQL 客户端。无实际作用。
    
- `sql_select_limit`
用于兼容 MySQL 客户端。无实际作用。

- `system_time_zone`
显示当前系统时区。不可更改。
    
- `time_zone`
用于设置当前会话的时区。时区会对某些时间函数的结果产生影响。
    
- `tx_isolation`
用于兼容 MySQL 客户端。无实际作用。
    
- `version`
用于兼容 MySQL 客户端。无实际作用。

- `performance_schema`
用于兼容 8.0.16及以上版本的MySQL JDBC。无实际作用。     
    
- `version_comment`
用于显示 Doris 的版本。不可更改。
    
- `wait_timeout`
用于设置空闲连接的连接时长。当一个空闲连接在该时长内与 Doris 没有任何交互，则 Doris 会主动断开这个链接。默认为 8 小时，单位为秒。

- `default_rowset_type`
用于设置计算节点存储引擎默认的存储格式。当前支持的存储格式包括：alpha/beta。

- `use_v2_rollup`
用于控制查询使用segment v2存储格式的rollup索引获取数据。该变量用于上线segment v2的时候，进行验证使用；其他情况，不建议使用。

- `rewrite_count_distinct_to_bitmap_hll`
是否将 bitmap 和 hll 类型的 count distinct 查询重写为 bitmap_union_count 和 hll_union_agg 。

- `prefer_join_method`
在选择 join 的具体实现方式是 broadcast join 还是 shuffle join 时，如果 broadcast join cost 和 shuffle join cost 相等时，优先选择哪种 join 方式。
目前该变量的可选值为"broadcast" 或者 "shuffle"。

- `allow_partition_column_nullable`
建表时是否允许分区列为 NULL。默认为 true，表示允许为 NULL。false 表示分区列必须被定义为 NOT NULL。

- `insert_visible_timeout_ms`
在执行 insert 语句时，导入动作(查询和插入)完成后，还需要等待事务提交，使数据可见。此参数控制等待数据可见的超时时间，默认为10000，最小为1000。
    
-  `enable_exchange_node_parallel_merge`
在一个排序的查询之中，一个上层节点接收下层节点有序数据时，会在 exchange node 上进行对应的排序来保证最终的数据是有序的。但是单线程进行多路数据归并时，如果数据量过大，会导致 exchange node 的单点的归并瓶颈。
Doris 在这部分进行了优化处理，如果下层的数据节点过多。exchange node 会启动多线程进行并行归并来加速排序过程。该参数默认为 False，即表示 exchange node 不采取并行的归并排序，来减少额外的 CPU 和内存消耗。

- `extract_wide_range_expr`
用于控制是否开启 「宽泛公因式提取」的优化。取值有两种：true 和 false 。默认情况下开启。

- `enable_fold_constant_by_be`
用于控制常量折叠的计算方式。默认是 `false`，即在 `FE` 进行计算；若设置为 `true`，则通过 `RPC` 请求经 `BE` 计算。 

- `cpu_resource_limit`
用于限制一个查询的资源开销。这是一个实验性质的功能。目前的实现是限制一个查询在单个节点上的 scan 线程数量。限制了 scan 线程数，从底层返回的数据速度变慢，从而限制了查询整体的计算资源开销。假设设置为 2，则一个查询在单节点上最多使用2个 scan 线程。
该参数会覆盖 `parallel_fragment_exec_instance_num` 的效果。即假设 `parallel_fragment_exec_instance_num` 设置为4，而该参数设置为2。则单个节点上的4个执行实例会共享最多2个扫描线程。
该参数会被 user property 中的 `cpu_resource_limit` 配置覆盖。
默认 -1，即不限制。

- `disable_join_reorder`
用于关闭所有系统自动的 join reorder 算法。取值有两种：true 和 false。默认行况下关闭，也就是采用系统自动的 join reorder 算法。设置为 true 后，系统会关闭所有自动排序的算法，采用 SQL 原始的表顺序，执行 join

- `return_object_data_as_binary` 
用于标识是否在 select 结果中返回 bitmap/hll 结果。在 select into outfile 语句中，如果导出文件格式为csv 则会将 bimap/hll 数据进行 base64编码，如果是 parquet 文件格式将会把数据作为 byte array 存储

- `block_encryption_mode` 
可以通过 block_encryption_mode 参数，控制块加密模式，默认值为：空。当使用AES算法加密时相当于`AES_128_ECB`, 当时用SM3算法加密时相当于`SM3_128_ECB` 可选值：
```text
  AES_128_ECB,
  AES_192_ECB,
  AES_256_ECB,
  AES_128_CBC,
  AES_192_CBC,
  AES_256_CBC,
  AES_128_CFB,
  AES_192_CFB,
  AES_256_CFB,
  AES_128_CFB1,
  AES_192_CFB1,
  AES_256_CFB1,
  AES_128_CFB8,
  AES_192_CFB8,
  AES_256_CFB8,
  AES_128_CFB128,
  AES_192_CFB128,
  AES_256_CFB128,
  AES_128_CTR,
  AES_192_CTR,
  AES_256_CTR,
  AES_128_OFB,
  AES_192_OFB,
  AES_256_OFB,
  SM4_128_ECB,
  SM4_128_CBC,
  SM4_128_CFB128,
  SM4_128_OFB,
  SM4_128_CTR,
```

- `enable_infer_predicate`
用于控制是否进行谓词推导。取值有两种：true 和 false。默认情况下关闭，系统不在进行谓词推导，采用原始的谓词进行相关操作。设置为 true 后，进行谓词扩展。

- `trim_tailing_spaces_for_external_table_query`
用于控制查询 Hive 外表时是否过滤掉字段末尾的空格。默认为 false。
