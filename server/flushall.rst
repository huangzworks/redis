.. _flushall:

FLUSHALL
=========

**FLUSHALL**

清空整个 Redis 服务器的数据(删除所有数据库的所有 key )。

此命令从不失败。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    尚未明确

**返回值：**
    总是返回 ``OK`` 。

::

    redis> DBSIZE            # 0 号数据库的 key 数量
    (integer) 9

    redis> SELECT 1          # 切换到 1 号数据库
    OK

    redis[1]> DBSIZE         # 1 号数据库的 key 数量
    (integer) 6

    redis[1]> flushall       # 清空所有数据库的所有 key 
    OK

    redis[1]> DBSIZE         # 不但 1 号数据库被清空了
    (integer) 0

    redis[1]> SELECT 0       # 0 号数据库(以及其他所有数据库)也一样
    OK

    redis> DBSIZE
    (integer) 0
