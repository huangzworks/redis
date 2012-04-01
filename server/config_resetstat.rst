.. _config_resetstat:

CONFIG RESETSTAT
=================

**CONFIG RESETSTAT**

重置 :ref:`INFO` 命令中的某些统计数据，包括：

- Keyspace hits (键空间命中次数)
- Keyspace misses (键空间不命中次数)
- Number of commands processed (执行命令的次数)
- Number of connections received (连接服务器的次数)
- Number of expired keys (过期key的数量)
- Number of rejected connections (被拒绝的连接数量)
- Latest fork(2) time(最后执行 fork(2) 的时间)
- The ``aof_delayed_fsync`` counter(``aof_delayed_fsync`` 计数器的值)

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    总是返回 ``OK`` 。

::

    # 重置前

    redis 127.0.0.1:6379> INFO
    # Server
    redis_version:2.5.3
    redis_git_sha1:d0407c2d
    redis_git_dirty:0
    arch_bits:32
    multiplexing_api:epoll
    gcc_version:4.6.3
    process_id:11095
    run_id:ef1f6b6c7392e52d6001eaf777acbe547d1192e2
    tcp_port:6379
    uptime_in_seconds:6
    uptime_in_days:0
    lru_clock:1205426

    # Clients
    connected_clients:1
    client_longest_output_list:0
    client_biggest_input_buf:0
    blocked_clients:0

    # Memory
    used_memory:331076
    used_memory_human:323.32K
    used_memory_rss:1568768
    used_memory_peak:293424
    used_memory_peak_human:286.55K
    used_memory_lua:16384
    mem_fragmentation_ratio:4.74
    mem_allocator:jemalloc-2.2.5

    # Persistence
    loading:0
    aof_enabled:0
    changes_since_last_save:0
    bgsave_in_progress:0
    last_save_time:1333260015
    last_bgsave_status:ok
    bgrewriteaof_in_progress:0

    # Stats
    total_connections_received:1
    total_commands_processed:0
    instantaneous_ops_per_sec:0
    rejected_connections:0
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:0
    pubsub_channels:0
    pubsub_patterns:0
    latest_fork_usec:0

    # Replication
    role:master
    connected_slaves:0

    # CPU
    used_cpu_sys:0.01
    used_cpu_user:0.00
    used_cpu_sys_children:0.00
    used_cpu_user_children:0.00

    # Keyspace
    db0:keys=20,expires=0


    # 重置

    redis 127.0.0.1:6379> CONFIG RESETSTAT
    OK

    
    # 重置后

    redis 127.0.0.1:6379> INFO
    # Server
    redis_version:2.5.3
    redis_git_sha1:d0407c2d
    redis_git_dirty:0
    arch_bits:32
    multiplexing_api:epoll
    gcc_version:4.6.3
    process_id:11095
    run_id:ef1f6b6c7392e52d6001eaf777acbe547d1192e2
    tcp_port:6379
    uptime_in_seconds:134
    uptime_in_days:0
    lru_clock:1205438

    # Clients
    connected_clients:1
    client_longest_output_list:0
    client_biggest_input_buf:0
    blocked_clients:0

    # Memory
    used_memory:331076
    used_memory_human:323.32K
    used_memory_rss:1568768
    used_memory_peak:330280
    used_memory_peak_human:322.54K
    used_memory_lua:16384
    mem_fragmentation_ratio:4.74
    mem_allocator:jemalloc-2.2.5

    # Persistence
    loading:0
    aof_enabled:0
    changes_since_last_save:0
    bgsave_in_progress:0
    last_save_time:1333260015
    last_bgsave_status:ok
    bgrewriteaof_in_progress:0

    # Stats
    total_connections_received:0
    total_commands_processed:1
    instantaneous_ops_per_sec:0
    rejected_connections:0
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:0
    pubsub_channels:0
    pubsub_patterns:0
    latest_fork_usec:0

    # Replication
    role:master
    connected_slaves:0

    # CPU
    used_cpu_sys:0.05
    used_cpu_user:0.02
    used_cpu_sys_children:0.00
    used_cpu_user_children:0.00

    # Keyspace
    db0:keys=20,expires=0
