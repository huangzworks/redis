.. _mget:

MGET
=====

**MGET key [key ...]**

返回所有(一个或多个)给定 ``key`` 的值。

如果给定的 ``key`` 里面，有某个 ``key`` 不存在，那么这个 ``key`` 返回特殊值 ``nil`` 。因此，该命令永不失败。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(N) , ``N`` 为给定 ``key`` 的数量。
                                        
**返回值：**
    一个包含所有给定 ``key`` 的值的列表。

::

    redis> SET redis redis.com
    OK

    redis> SET mongodb mongodb.org
    OK

    redis> MGET redis mongodb
    1) "redis.com"
    2) "mongodb.org"

    redis> MGET redis mongodb mysql     # 不存在的 mysql 返回 nil
    1) "redis.com"
    2) "mongodb.org"
    3) (nil)
