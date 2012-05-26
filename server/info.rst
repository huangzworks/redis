.. _info:

INFO
======

**INFO**

返回关于 Redis 服务器的各种信息和统计值。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    具体请参见下面的测试代码。

::

    redis> INFO
    # Server
    redis_version:2.5.9
    redis_git_sha1:473f3090
    redis_git_dirty:0
    os:Linux 3.3.7-1-ARCH i686
    arch_bits:32
    multiplexing_api:epoll
    gcc_version:4.7.0
    process_id:8104
    run_id:bc9e20c6f0aac67d0d396ab950940ae4d1479ad1
    tcp_port:6379
    uptime_in_seconds:7
    uptime_in_days:0
    lru_clock:1680564

    # Clients
    connected_clients:1
    client_longest_output_list:0
    client_biggest_input_buf:0
    blocked_clients:0

    # Memory
    used_memory:439304
    used_memory_human:429.01K
    used_memory_rss:13897728
    used_memory_peak:401776
    used_memory_peak_human:392.36K
    used_memory_lua:20480
    mem_fragmentation_ratio:31.64
    mem_allocator:jemalloc-3.0.0

    # Persistence
    loading:0
    rdb_changes_since_last_save:0
    rdb_bgsave_in_progress:0
    rdb_last_save_time:1338011402
    rdb_last_bgsave_status:ok
    rdb_last_bgsave_time_sec:-1
    rdb_current_bgsave_time_sec:-1
    aof_enabled:0
    aof_rewrite_in_progress:0
    aof_rewrite_scheduled:0
    aof_last_rewrite_time_sec:-1
    aof_current_rewrite_time_sec:-1

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
    used_cpu_sys:0.03
    used_cpu_user:0.01
    used_cpu_sys_children:0.00
    used_cpu_user_children:0.00

    # Keyspace
