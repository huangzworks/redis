.. Redis命令参考简体中文版 documentation master file, created by
   sphinx-quickstart on Tue Oct 25 17:56:34 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 命令参考
=================

本文是 Redis Command Reference(`redis.io/commands <http://redis.io/commands>`_)的简体中文翻译版，
原文十一个部分、共一百多个命令已经全部翻译完毕。

本文所有示例代码均经过 Redis 2.6 版本测试，质量保证。


目录(使用 CTRL + F 快速查找)：
----------------------------------

+-----------------------------------+-------------------------------------------+---------------------------------------+-----------------------------------+
| - Key(键)                         | - String(字符串)                          | - Hash(哈希表)                        | - List(列表)                      |
|                                   |                                           |                                       |                                   |
|   - :doc:`key/del`                |   - :doc:`string/set`                     |   - :doc:`hash/hset`                  |   - :doc:`list/lpush`             |
|   - :doc:`key/keys`               |   - :doc:`string/setnx`                   |   - :doc:`hash/hsetnx`                |   - :doc:`list/lpushx`            |
|   - :doc:`key/randomkey`          |   - :doc:`string/setex`                   |   - :doc:`hash/hmset`                 |   - :doc:`list/rpush`             |
|   - :doc:`key/ttl`                |   - :doc:`string/psetex`                  |   - :doc:`hash/hget`                  |   - :doc:`list/rpushx`            |
|   - :doc:`key/pttl`               |   - :doc:`string/setrange`                |   - :doc:`hash/hmget`                 |   - :doc:`list/lpop`              |
|   - :doc:`key/exists`             |   - :doc:`string/mset`                    |   - :doc:`hash/hgetall`               |   - :doc:`list/rpop`              |
|   - :doc:`key/move`               |   - :doc:`string/msetnx`                  |   - :doc:`hash/hdel`                  |   - :doc:`list/blpop`             |
|   - :doc:`key/rename`             |   - :doc:`string/append`                  |   - :doc:`hash/hlen`                  |   - :doc:`list/brpop`             |
|   - :doc:`key/renamenx`           |   - :doc:`string/get`                     |   - :doc:`hash/hexists`               |   - :doc:`list/llen`              |
|   - :doc:`key/type`               |   - :doc:`string/mget`                    |   - :doc:`hash/hincrby`               |   - :doc:`list/lrange`            |
|   - :doc:`key/expire`             |   - :doc:`string/getrange`                |   - :doc:`hash/hincrbyfloat`          |   - :doc:`list/lrem`              |
|   - :doc:`key/pexpire`            |   - :doc:`string/getset`                  |   - :doc:`hash/hkeys`                 |   - :doc:`list/lset`              |
|   - :doc:`key/expireat`           |   - :doc:`string/strlen`                  |   - :doc:`hash/hvals`                 |   - :doc:`list/ltrim`             |
|   - :doc:`key/pexpireat`          |   - :doc:`string/decr`                    |                                       |   - :doc:`list/lindex`            |
|   - :doc:`key/persist`            |   - :doc:`string/decrby`                  |                                       |   - :doc:`list/linsert`           |
|   - :doc:`key/sort`               |   - :doc:`string/incr`                    |                                       |   - :doc:`list/rpoplpush`         |
|   - :doc:`key/object`             |   - :doc:`string/incrby`                  |                                       |   - :doc:`list/brpoplpush`        |
|   - :doc:`key/migrate`            |   - :doc:`string/incrbyfloat`             |                                       |                                   |
|   - :doc:`key/dump`               |   - :doc:`string/setbit`                  |                                       |                                   |
|   - :doc:`key/restore`            |   - :doc:`string/getbit`                  |                                       |                                   |
|                                   |   - :doc:`string/bitop`                   |                                       |                                   |
|                                   |   - :doc:`string/bitcount`                |                                       |                                   |
|                                   |                                           |                                       |                                   |
+-----------------------------------+-------------------------------------------+---------------------------------------+-----------------------------------+
| |                                 | |                                         | |                                     | |                                 |
| - Set(集合)                       | - 有序集(Sorted set)                      | - Pub/Sub(发布/订阅)                  | - Transaction(事务)               |
|                                   |                                           |                                       |                                   |
|   - :doc:`set/sadd`               |   - :doc:`sorted_set/zadd`                |   - :doc:`pub_sub/publish`            |   - :doc:`transaction/watch`      |
|   - :doc:`set/srem`               |   - :doc:`sorted_set/zrem`                |   - :doc:`pub_sub/subscribe`          |   - :doc:`transaction/unwatch`    |
|   - :doc:`set/smembers`           |   - :doc:`sorted_set/zcard`               |   - :doc:`pub_sub/psubscribe`         |   - :doc:`transaction/multi`      |
|   - :doc:`set/sismember`          |   - :doc:`sorted_set/zcount`              |   - :doc:`pub_sub/unsubscribe`        |   - :doc:`transaction/discard`    | 
|   - :doc:`set/scard`              |   - :doc:`sorted_set/zscore`              |   - :doc:`pub_sub/punsubscribe`       |   - :doc:`transaction/exec`       |
|   - :doc:`set/smove`              |   - :doc:`sorted_set/zincrby`             |                                       |                                   |
|   - :doc:`set/spop`               |   - :doc:`sorted_set/zrange`              |                                       |                                   |
|   - :doc:`set/srandmember`        |   - :doc:`sorted_set/zrevrange`           |                                       |                                   |
|   - :doc:`set/sinter`             |   - :doc:`sorted_set/zrangebyscore`       |                                       |                                   |
|   - :doc:`set/sinterstore`        |   - :doc:`sorted_set/zrevrangebyscore`    |                                       |                                   |
|   - :doc:`set/sunion`             |   - :doc:`sorted_set/zrank`               |                                       |                                   |
|   - :doc:`set/sunionstore`        |   - :doc:`sorted_set/zrevrank`            |                                       |                                   |
|   - :doc:`set/sdiff`              |   - :doc:`sorted_set/zremrangebyrank`     |                                       |                                   |
|   - :doc:`set/sdiffstore`         |   - :doc:`sorted_set/zremrangebyscore`    |                                       |                                   |
|                                   |   - :doc:`sorted_set/zinterstore`         |                                       |                                   |
|                                   |   - :doc:`sorted_set/zunionstore`         |                                       |                                   |
|                                   |                                           |                                       |                                   |
+-----------------------------------+-------------------------------------------+---------------------------------------+-----------------------------------+
| |                                 | |                                         | |                                     |                                   |
| - Script(脚本)                    | - Connection(连接)                        | - Server(服务器)                      |                                   |
|                                   |                                           |                                       |                                   |
|   - :doc:`script/eval`            |   - :doc:`connection/auth`                |   - :doc:`server/time`                |                                   |
|   - :doc:`script/evalsha`         |   - :doc:`connection/ping`                |   - :doc:`server/dbsize`              |                                   |
|   - :doc:`script/script_load`     |   - :doc:`connection/select`              |   - :doc:`server/bgrewriteaof`        |                                   |
|   - :doc:`script/script_exists`   |   - :doc:`connection/echo`                |   - :doc:`server/bgsave`              |                                   |
|   - :doc:`script/script_kill`     |   - :doc:`connection/quit`                |   - :doc:`server/save`                |                                   |
|   - :doc:`script/script_flush`    |                                           |   - :doc:`server/lastsave`            |                                   |
|                                   |                                           |   - :doc:`server/slaveof`             |                                   |
|                                   |                                           |   - :doc:`server/flushall`            |                                   |
|                                   |                                           |   - :doc:`server/flushdb`             |                                   |
|                                   |                                           |   - :doc:`server/shutdown`            |                                   |
|                                   |                                           |   - :doc:`server/slowlog`             |                                   |
|                                   |                                           |   - :doc:`server/info`                |                                   |
|                                   |                                           |   - :doc:`server/config_get`          |                                   |
|                                   |                                           |   - :doc:`server/config_set`          |                                   |
|                                   |                                           |   - :doc:`server/config_resetstat`    |                                   |
|                                   |                                           |   - :doc:`server/debug_object`        |                                   |
|                                   |                                           |   - :doc:`server/debug_segfault`      |                                   |
|                                   |                                           |   - :doc:`server/monitor`             |                                   |
|                                   |                                           |   - :doc:`server/sync`                |                                   |
|                                   |                                           |                                       |                                   |
+-----------------------------------+-------------------------------------------+---------------------------------------+-----------------------------------+

下载离线版本
------------------


`HTML 格式 <http://media.readthedocs.org/htmlzip/redis/latest/redis.zip>`_ ， PDF 格式(因为中文编码问题，暂不支持)。

注意，因为文档总是在不断地更新和修正当中，请定期下载最新的离线文档，确保获得最好的阅读体验。

关于
-------

查看最新工作进度、加入本项目或联系译者，请阅读\ :doc:`readme`\ 页面。

评论
------
