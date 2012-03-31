.. Redis命令参考简体中文版 documentation master file, created by
   sphinx-quickstart on Tue Oct 25 17:56:34 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis命令参考简体中文版
=============================================

本文是\ `《Redis Command Reference》 <http://redis.io/commands>`_\ 的简体中文翻译版，原文共十个部分的\ **所有命令均已翻译完毕** 。

本文所有示例代码均经过 Redis 2.4.4 版本测试，质量保证。 提交翻译问题、加入本项目或联系译者，请阅读\ :ref:`readme`\ 页面。

Key(键)
-------------------

:ref:`DEL` ,
:ref:`KEYS` ,
:ref:`RANDOMKEY` ,
:ref:`TTL`,
:ref:`PTTL` ,
:ref:`EXISTS` ,
:ref:`MOVE` ,
:ref:`RENAME` ,
:ref:`RENAMENX` ,
:ref:`TYPE` ,
:ref:`EXPIRE` ,
:ref:`PEXPIRE` ,
:ref:`EXPIREAT` ,
:ref:`OBJECT` ,
:ref:`PEXPIREAT` ,
:ref:`PERSIST` ,
:ref:`SORT`

String(字符串)
-------------------

:ref:`SET` ,
:ref:`SETNX` ,
:ref:`SETEX` ,
:ref:`PSETEX` ,
:ref:`SETRANGE` ,
:ref:`MSET` ,
:ref:`MSETNX` ,
:ref:`APPEND` ,
:ref:`GET` ,
:ref:`MGET` ,
:ref:`GETRANGE` ,
:ref:`GETSET` ,
:ref:`STRLEN` ,
:ref:`DECR` ,
:ref:`DECRBY` ,
:ref:`INCR` ,
:ref:`INCRBY` ,
:ref:`INCRBYFLOAT` ,
:ref:`SETBIT` ,
:ref:`GETBIT` 

Hash(哈希表)
-------------------

:ref:`HSET` ,
:ref:`HSETNX` ,
:ref:`HMSET` ,
:ref:`HGET` ,
:ref:`HMGET` ,
:ref:`HGETALL` ,
:ref:`HDEL` ,
:ref:`HLEN` ,
:ref:`HEXISTS` ,
:ref:`HINCRBY` ,
:ref:`HINCRBYFLOAT` ,
:ref:`HKEYS`,
:ref:`HVALS`

List(列表)
-------------------

:ref:`LPUSH` ,
:ref:`LPUSHX` ,
:ref:`RPUSH` ,
:ref:`RPUSHX` ,
:ref:`LPOP` ,
:ref:`RPOP` ,
:ref:`BLPOP` ,
:ref:`BRPOP` ,
:ref:`LLEN` ,
:ref:`LRANGE` ,
:ref:`LREM` ,
:ref:`LSET` ,
:ref:`LTRIM` ,
:ref:`LINDEX` ,
:ref:`LINSERT` ,
:ref:`RPOPLPUSH` ,
:ref:`BRPOPLPUSH` 

Set(集合)
-------------------------

:ref:`SADD` ,
:ref:`SREM` ,
:ref:`SMEMBERS` ,
:ref:`SISMEMBER` ,
:ref:`SCARD` ,
:ref:`SMOVE` ,
:ref:`SPOP` ,
:ref:`SRANDMEMBER` ,
:ref:`SINTER` ,
:ref:`SINTERSTORE` ,
:ref:`SUNION` ,
:ref:`SUNIONSTORE` ,
:ref:`SDIFF` ,
:ref:`SDIFFSTORE` 

Sorted Set(有序集)
-------------------------

:ref:`ZADD` ,
:ref:`ZREM` ,
:ref:`ZCARD` ,
:ref:`ZCOUNT` ,
:ref:`ZSCORE` ,
:ref:`ZINCRBY` ,
:ref:`ZRANGE` ,
:ref:`ZREVRANGE` ,
:ref:`ZRANGEBYSCORE` ,
:ref:`ZREVRANGEBYSCORE` ,
:ref:`ZRANK` ,
:ref:`ZREVRANK` ,
:ref:`ZREMRANGEBYRANK` ,
:ref:`ZREMRANGEBYSCORE` ,
:ref:`ZINTERSTORE` ,
:ref:`ZUNIONSTORE`

Pub/Sub(发布/订阅)
---------------------------------

:ref:`publish` ,
:ref:`subscribe` ,
:ref:`psubscribe` ,
:ref:`unsubscribe` ,
:ref:`punsubscribe` 

Transaction(事务)
--------------------------

:ref:`watch` ,
:ref:`unwatch` ,
:ref:`multi` ,
:ref:`exec` ,
:ref:`discard` 

Connection(连接)
----------------------

:ref:`auth` ,
:ref:`ping` ,
:ref:`select` ,
:ref:`echo` ,
:ref:`quit` 

Server(服务器)
---------------------

:ref:`time` ,
:ref:`dbsize` ,
:ref:`bgrewriteaof` ,
:ref:`bgsave` ,
:ref:`save` ,
:ref:`lastsave` ,
:ref:`slaveof` ,
:ref:`flushall` ,
:ref:`flushdb` ,
:ref:`shutdown` ,
:ref:`slowlog` ,
:ref:`info` ,
:ref:`config_get` ,
:ref:`config_set` ,
:ref:`config_resetstat` ,
:ref:`debug_object` ,
:ref:`debug_segfault` ,
:ref:`monitor` ,
:ref:`sync`

Script(脚本)
--------------------

:ref:`eval` ,
:ref:`script_flush` ,
:ref:`script_load` ,
:ref:`script_exists` ,
:ref:`script_kill` 
