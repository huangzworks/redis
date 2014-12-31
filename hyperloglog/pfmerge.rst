.. _pfmerge:

PFMERGE
===============

**PFMERGE destkey sourcekey [sourcekey ...]**

..
    Merge multiple HyperLogLog values into an unique value 
    that will approximate the cardinality of the union of the observed Sets of the source HyperLogLog structures.

将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ，
合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。

..
    The computed merged HyperLogLog is set to the destination variable, 
    which is created if does not exist (defauling to an empty HyperLogLog).

合并得出的 HyperLogLog 会被储存在 ``destkey`` 键里面，
如果该键并不存在，
那么命令在执行之前，
会先为该键创建一个空的 HyperLogLog 。

**可用版本：**
    >= 2.8.9

**时间复杂度：**
    O(N) ，
    其中 N 为被合并的 HyperLogLog 数量，
    不过这个命令的常数复杂度比较高。

..  O(N) to merge N HyperLogLogs, but with high constant times.

**返回值：**
    字符串回复：返回 ``OK`` 。

..  Simple string reply: The command just returns OK.

::

    redis> PFADD  nosql  "Redis"  "MongoDB"  "Memcached"
    (integer) 1

    redis> PFADD  RDBMS  "MySQL" "MSSQL" "PostgreSQL"
    (integer) 1

    redis> PFMERGE  databases  nosql  RDBMS
    OK

    redis> PFCOUNT  databases
    (integer) 6
