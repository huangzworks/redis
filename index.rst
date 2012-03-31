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

:doc:`key/del` ,
:doc:`key/keys` ,
:doc:`key/randomkey` ,
:doc:`key/ttl`,
:doc:`key/pttl` ,
:doc:`key/exists` ,
:doc:`key/move` ,
:doc:`key/rename` ,
:doc:`key/renamenx` ,
:doc:`key/type` ,
:doc:`key/expire` ,
:doc:`key/pexpire` ,
:doc:`key/expireat` ,
:doc:`key/object` ,
:doc:`key/pexpireat` ,
:doc:`key/persist` ,
:doc:`key/sort`

String(字符串)
-------------------

:doc:`string/set` ,
:doc:`string/setnx` ,
:doc:`string/setex` ,
:doc:`string/psetex` ,
:doc:`string/setrange` ,
:doc:`string/mset` ,
:doc:`string/msetnx` ,
:doc:`string/append` ,
:doc:`string/get` ,
:doc:`string/mget` ,
:doc:`string/getrange` ,
:doc:`string/getset` ,
:doc:`string/strlen` ,
:doc:`string/decr` ,
:doc:`string/decrby` ,
:doc:`string/incr` ,
:doc:`string/incrby` ,
:doc:`string/incrbyfloat` ,
:doc:`string/setbit` ,
:doc:`string/getbit` 

Hash(哈希表)
-------------------

:doc:`hash/hset` ,
:doc:`hash/hsetnx` ,
:doc:`hash/hmset` ,
:doc:`hash/hget` ,
:doc:`hash/hmget` ,
:doc:`hash/hgetall` ,
:doc:`hash/hdel` ,
:doc:`hash/hlen` ,
:doc:`hash/hexists` ,
:doc:`hash/hincrby` ,
:doc:`hash/hincrbyfloat` ,
:doc:`hash/hkeys`,
:doc:`hash/hvals`

List(列表)
-------------------

:doc:`list/lpush` ,
:doc:`list/lpushx` ,
:doc:`list/rpush` ,
:doc:`list/rpushx` ,
:doc:`list/lpop` ,
:doc:`list/rpop` ,
:doc:`list/blpop` ,
:doc:`list/brpop` ,
:doc:`list/llen` ,
:doc:`list/lrange` ,
:doc:`list/lrem` ,
:doc:`list/lset` ,
:doc:`list/ltrim` ,
:doc:`list/lindex` ,
:doc:`list/linsert` ,
:doc:`list/rpoplpush` ,
:doc:`list/brpoplpush` 

Set(集合)
-------------------------

:doc:`set/sadd` ,
:doc:`set/srem` ,
:doc:`set/smembers` ,
:doc:`set/sismember` ,
:doc:`set/scard` ,
:doc:`set/smove` ,
:doc:`set/spop` ,
:doc:`set/srandmember` ,
:doc:`set/sinter` ,
:doc:`set/sinterstore` ,
:doc:`set/sunion` ,
:doc:`set/sunionstore` ,
:doc:`set/sdiff` ,
:doc:`set/sdiffstore` 

Sorted Set(有序集)
-------------------------

:doc:`sorted_set/zadd` ,
:doc:`sorted_set/zrem` ,
:doc:`sorted_set/zcard` ,
:doc:`sorted_set/zcount` ,
:doc:`sorted_set/zscore` ,
:doc:`sorted_set/zincrby` ,
:doc:`sorted_set/zrange` ,
:doc:`sorted_set/zrevrange` ,
:doc:`sorted_set/zrangebyscore` ,
:doc:`sorted_set/zrevrangebyscore` ,
:doc:`sorted_set/zrank` ,
:doc:`sorted_set/zrevrank` ,
:doc:`sorted_set/zremrangebyrank` ,
:doc:`sorted_set/zremrangebyscore` ,
:doc:`sorted_set/zinterstore` ,
:doc:`sorted_set/zunionstore`

Pub/Sub(发布/订阅)
---------------------------------

:doc:`pub_sub/publish` ,
:doc:`pub_sub/subscribe` ,
:doc:`pub_sub/psubscribe` ,
:doc:`pub_sub/unsubscribe` ,
:doc:`pub_sub/punsubscribe` 

Transaction(事务)
--------------------------

:doc:`transaction/watch` ,
:doc:`transaction/unwatch` ,
:doc:`transaction/multi` ,
:doc:`transaction/exec` ,
:doc:`transaction/discard` 

Connection(连接)
----------------------

:doc:`connection/auth` ,
:doc:`connection/ping` ,
:doc:`connection/select` ,
:doc:`connection/echo` ,
:doc:`connection/quit` 

Server(服务器)
---------------------

:doc:`server/time` ,
:doc:`server/dbsize` ,
:doc:`server/bgrewriteaof` ,
:doc:`server/bgsave` ,
:doc:`server/save` ,
:doc:`server/lastsave` ,
:doc:`server/slaveof` ,
:doc:`server/flushall` ,
:doc:`server/flushdb` ,
:doc:`server/shutdown` ,
:doc:`server/slowlog` ,
:doc:`server/info` ,
:doc:`server/config_get` ,
:doc:`server/config_set` ,
:doc:`server/config_resetstat` ,
:doc:`server/debug_object` ,
:doc:`server/debug_segfault` ,
:doc:`server/monitor` ,
:doc:`server/sync`

Script(脚本)
--------------------

:doc:`script/eval` ,
:doc:`script/script_flush` ,
:doc:`script/script_load` ,
:doc:`script/script_exists` ,
:doc:`script/script_kill` 
