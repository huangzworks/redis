.. _info:

INFO
======

**INFO**

返回关于 Redis 服务器的各种信息和统计值。

**时间复杂度：**
    O(1)

**返回值：**
    具体请参见下面的测试代码。

::

    redis> INFO
    redis_version:2.4.4             # Redis 的版本
    redis_git_sha1:00000000
    redis_git_dirty:0
    arch_bits:32
    multiplexing_api:epoll
    process_id:903                  # 当前 Redis 服务器进程 id
    uptime_in_seconds:24612         # 运行时间(以秒计算)
    uptime_in_days:0                # 运行时间(以日计算)
    lru_clock:283730            
    used_cpu_sys:3.38
    used_cpu_user:2.15
    used_cpu_sys_children:0.11
    used_cpu_user_children:0.00
    connected_clients:1             # 连接的客户端数量
    connected_slaves:0              # 从属服务器的数量
    client_longest_output_list:0    
    client_biggest_input_buf:0
    blocked_clients:0
    used_memory:557304              # Redis 分配的内存总量
    used_memory_human:544.24K       
    used_memory_rss:17879040        # Redis 分配的内存总量(包括内存碎片)
    used_memory_peak:565904
    used_memory_peak_human:552.64K
    mem_fragmentation_ratio:32.08   # 内存碎片比率
    mem_allocator:jemalloc-2.2.5    # 目前使用的内存分配库
    loading:0   
    aof_enabled:0
    changes_since_last_save:2       # 上次保存数据库之后，执行命令的次数
    bgsave_in_progress:0            # 后台进行中的 save 操作的数量
    last_save_time:1324042687       # 最后一次成功保存的时间点，以 UNIX 时间戳格式显示
    bgrewriteaof_in_progress:0      # 后台进行中的 aof 文件修改操作的数量
    total_connections_received:16   # 运行以来连接过的客户端的总数量
    total_commands_processed:87     # 运行以来执行过的命令的总数量
    expired_keys:0                  # 运行以来过期的 key 的数量
    evicted_keys:0
    keyspace_hits:14                # 命中 key 的次数
    keyspace_misses:14              # 不命中 key 的次数
    pubsub_channels:0               # 当前使用中的频道数量
    pubsub_patterns:0               # 当前使用的模式的数量
    latest_fork_usec:314
    vm_enabled:0                    # 是否开启了 vm
    role:master
    db0:keys=6,expires=0            # 各个数据库的 key 的数量，以及带有生存期的 key 的数量
    db1:keys=6,expires=0
    db2:keys=1,expires=0



