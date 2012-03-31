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

    # 重置前的部分数据

    redis> INFO
    # ...
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:5
    # ...

    # 重置

    redis> CONFIG RESETSTAT
    OK

    # 重置后的部分数据

    redis> INFO
    # ...
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:0
    pubsub_channels:0
    # ...
